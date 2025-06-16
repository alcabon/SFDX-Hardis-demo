```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>afklm.salesforce</groupId>
	<artifactId>salesforcemaven</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<dependencies>
		<!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
		<dependency>
			<groupId>com.google.code.gson</groupId>
			<artifactId>gson</artifactId>
			<version>2.10</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.force.api/force-wsc -->
		<dependency>
			<groupId>com.force.api</groupId>
			<artifactId>force-wsc</artifactId>
			<version>59.0.0</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.force.api/force-partner-api -->
		<dependency>
			<groupId>com.force.api</groupId>
			<artifactId>force-partner-api</artifactId>
			<version>59.0.0</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.force.api/force-metadata-api -->
		<dependency>
			<groupId>com.force.api</groupId>
			<artifactId>force-metadata-api</artifactId>
			<version>59.0.0</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.force.api/force-apex-api -->
		<dependency>
			<groupId>com.force.api</groupId>
			<artifactId>force-apex-api</artifactId>
			<version>59.0.0</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-csv -->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-csv</artifactId>
			<version>1.8</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.univocity/univocity-parsers -->
		<dependency>
			<groupId>com.univocity</groupId>
			<artifactId>univocity-parsers</artifactId>
			<version>2.9.1</version>
		</dependency>
		<dependency>
			<groupId>com.github.nawforce</groupId>
			<artifactId>apex-parser</artifactId>
			<version>2.12.0</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>3.12.0</version>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-csv</artifactId>
			<version>1.9.0</version>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-text</artifactId>
			<version>1.10.0</version>
		</dependency>
			<dependency>
			<groupId>org.json</groupId>
			<artifactId>json</artifactId>
			<version>20240303</version>
		</dependency>
	</dependencies>
</project>
```

```java
package afklm.salesforce;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import org.json.JSONObject;
import com.sforce.soap.metadata.MetadataConnection;
import com.sforce.soap.partner.PartnerConnection;
import com.sforce.ws.ConnectionException;
import com.sforce.ws.ConnectorConfig;

public class SalesforceConnection {
	
	public static MetadataConnection createMetadataConnection(PartnerConnection partnerConnection)
			throws ConnectionException {

		ConnectorConfig config = new ConnectorConfig();
		config.setSessionId(partnerConnection.getConfig().getSessionId());
		config.setServiceEndpoint(partnerConnection.getConfig().getServiceEndpoint().replace("/u/", "/m/"));

		return new MetadataConnection(config);
	}
	
	public static PartnerConnection createPartnerConnection(String accessToken, String instanceUrl) throws Exception {
		ConnectorConfig config = new ConnectorConfig();
		config.setSessionId(accessToken);
		config.setServiceEndpoint(instanceUrl + "/services/Soap/u/61.0");

		return new PartnerConnection(config);
	}

	public static String executeCommand(String alias) throws Exception {
		ProcessBuilder processBuilder = new ProcessBuilder("sf.cmd", "org", "display", "-u", alias, "--json");
		Process process = processBuilder.start();

		// Process process = Runtime.getRuntime().exec("sf org display -u " + alias + "
		// --json");

		StringBuilder output = new StringBuilder();
		BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));

		String line;
		while ((line = reader.readLine()) != null) {
			output.append(line);
		}
		int exitCode = process.waitFor();
		if (exitCode != 0) {
			throw new Exception("La commande a échoué avec le code de sortie : " + exitCode);
		}
		return output.toString();
	}

	public static PartnerConnection login(String alias) {
		try {
			String jsonOutput = executeCommand(alias);

			JSONObject jsonObject = new JSONObject(jsonOutput);

			String accessToken = jsonObject.getJSONObject("result").getString("accessToken");
			String instanceUrl = jsonObject.getJSONObject("result").getString("instanceUrl");

			// Utilisez ces informations pour créer votre connexion partner
			PartnerConnection connection = createPartnerConnection(accessToken, instanceUrl);
			System.out.println("getOrganizationName:\t" + connection.getUserInfo().getOrganizationName());
			System.out.println("getUserFullName:\t" + connection.getUserInfo().getUserFullName());
			System.out.println("getUserEmail:\t\t" + connection.getUserInfo().getUserEmail());

			return connection;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}
}
```

```java
package afklm.salesforce;

import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;

import com.sforce.soap.metadata.DescribeMetadataObject;
import com.sforce.soap.metadata.DescribeMetadataResult;
import com.sforce.soap.metadata.FileProperties;
import com.sforce.soap.metadata.ListMetadataQuery;
import com.sforce.soap.metadata.MetadataConnection;
import com.sforce.soap.metadata.Package;
import com.sforce.soap.metadata.PackageTypeMembers;
import com.sforce.soap.partner.PartnerConnection;
import com.sforce.soap.partner.QueryResult;
import com.sforce.soap.partner.sobject.SObject;
import com.sforce.ws.ConnectionException;

public class MetadataRetriever2 {

	private static final int MAX_ARTIFACTS_PER_PACKAGE = 10000;
	private PartnerConnection partnerConnection;
	private MetadataConnection metadataConnection;

	// Définition des types de métadonnées clés pour les profils/ensembles de
	// permissions
	private final Map<String, List<String>> keyMetadataTypes = new HashMap<>();

	// Types critiques qui doivent TOUJOURS être inclus même sans membres listés
	private final Set<String> criticalTypes = new HashSet<>(
			Arrays.asList("Profile", "PermissionSet", "CustomObject", "ApexClass", "CustomApplication"));

	public MetadataRetriever2() {
		// Initialisation des types de métadonnées clés.
		keyMetadataTypes.put("CustomObject", Arrays.asList("*"));
		keyMetadataTypes.put("CustomField", new ArrayList<>()); // Ne supporte pas le générique
		keyMetadataTypes.put("ApexClass", Arrays.asList("*"));
		keyMetadataTypes.put("ApexPage", Arrays.asList("*"));
		keyMetadataTypes.put("CustomApplication", Arrays.asList("*"));
		keyMetadataTypes.put("RecordType", new ArrayList<>()); // Ne supporte pas le générique
		keyMetadataTypes.put("CustomTab", Arrays.asList("*"));
		keyMetadataTypes.put("ExternalDataSource", Arrays.asList("*"));
		keyMetadataTypes.put("Flow", Arrays.asList("*"));
		keyMetadataTypes.put("CustomPermission", Arrays.asList("*"));
		keyMetadataTypes.put("CustomMetadata", Arrays.asList("*"));
		keyMetadataTypes.put("Profile", Arrays.asList("*")); // Les profils sont cruciaux
		keyMetadataTypes.put("PermissionSet", Arrays.asList("*")); // Les ensembles de permissions aussi
	}

	public static void main(String[] args) {
		args = new String[] { "myalias" }; // Décommenter pour tester avec un alias fixe
		if (args.length < 1) {
			System.err.println("Usage: java -cp \"bin:lib/*\" afklm.salesforce.MetadataRetriever <alias_salesforce>");
			System.err.println("Exemple: java -cp \"bin:lib/*\" afklm.salesforce.MetadataRetriever devorg");
			System.exit(1);
		}

		MetadataRetriever2 retriever = new MetadataRetriever2();
		retriever.execute(args[0]);
	}

	public void execute(String alias) {
		try {
			// 0) Se connecter à Salesforce
			login(alias);

			// 1) Récupérer les types de métadonnées disponibles
			List<DescribeMetadataObject> metadataTypes = retrieveMetadataTypes();
			System.out.println("Types de métadonnées disponibles (" + metadataTypes.size() + "):");
			for (DescribeMetadataObject type : metadataTypes) {
				System.out.println("- " + type.getXmlName() + " (directory: " + type.getDirectoryName() + ", suffix: "
						+ type.getSuffix() + ", inFolder: " + type.isInFolder() + ")");
			}

			// 2) Créer des packages XML avec gestion de la limite et groupement
			createMetadataPackages(metadataTypes);

		} catch (ConnectionException e) {
			System.err.println("Erreur de connexion Salesforce: " + e.getMessage());
			e.printStackTrace();
		} catch (Exception e) {
			System.err.println("Une erreur inattendue est survenue: " + e.getMessage());
			e.printStackTrace();
		} finally {
			if (partnerConnection != null) {
				try {
					partnerConnection.logout();
					System.out.println("Déconnexion réussie.");
				} catch (ConnectionException e) {
					System.err.println("Erreur lors de la déconnexion: " + e.getMessage());
				}
			}
		}
	}

	private void login(String alias) throws ConnectionException {
		try {
			partnerConnection = SalesforceConnection.login(alias);

			if (partnerConnection == null) {
				throw new ConnectionException("Impossible de se connecter avec l'alias: " + alias);
			}

			metadataConnection = SalesforceConnection.createMetadataConnection(partnerConnection);

			System.out.println("Connexion Salesforce réussie!");

		} catch (Exception e) {
			throw new ConnectionException("Erreur lors de la connexion: " + e.getMessage(), e);
		}
	}

	private List<DescribeMetadataObject> retrieveMetadataTypes() throws ConnectionException {
		if (metadataConnection == null) {
			throw new ConnectionException("La connexion metadata n'est pas initialisée.");
		}

		DescribeMetadataResult results = metadataConnection.describeMetadata(58.0);
		return Arrays.asList(results.getMetadataObjects());
	}

	private void createMetadataPackages(List<DescribeMetadataObject> availableMetadataTypes)
			throws ConnectionException, IOException {

		// Collecter les membres pour tous les types clés avec requêtes SOQL améliorées
		Map<String, List<String>> allMembersCollected = collectMembersWithSOQL();

		// Créer le package principal avec les métadonnées clés
		createMainKeyPackage(allMembersCollected);

		// Créer les packages secondaires pour les autres métadonnées
		createSecondaryPackages(availableMetadataTypes, allMembersCollected);
	}

	// Méthode pour récupérer les profiles via requête SOQL
	private List<String> getProfileMembersByQuery() throws ConnectionException {
		List<String> profileNames = new ArrayList<>();

		try {
			if (!canQueryObject("Profile")) {
				System.out.println("    ⚠ Pas d'accès à l'objet Profile, utilisation du wildcard");
				return profileNames; // Retourne liste vide pour utiliser wildcard
			}

			String soqlQuery = "SELECT Name FROM Profile WHERE Name != null ORDER BY Name";
			System.out.println("    Exécution de la requête SOQL : " + soqlQuery);

			QueryResult queryResult = partnerConnection.query(soqlQuery);

			if (queryResult.getSize() > 0) {
				SObject[] records = queryResult.getRecords();

				for (SObject record : records) {
					String profileName = (String) record.getField("Name");
					if (profileName != null && !profileName.trim().isEmpty()) {
						profileNames.add(profileName);
					}
				}

				// Gérer la pagination si nécessaire
				while (!queryResult.isDone()) {
					queryResult = partnerConnection.queryMore(queryResult.getQueryLocator());
					SObject[] moreRecords = queryResult.getRecords();

					for (SObject record : moreRecords) {
						String profileName = (String) record.getField("Name");
						if (profileName != null && !profileName.trim().isEmpty()) {
							profileNames.add(profileName);
						}
					}
				}

				System.out.println("    → " + profileNames.size() + " profiles trouvés via SOQL");

			} else {
				System.out.println("    → Aucun profile trouvé via SOQL");
			}

		} catch (ConnectionException e) {
			System.err.println("    Erreur lors de la requête SOQL pour les profiles: " + e.getMessage());
			// Retourner liste vide pour utiliser wildcard en fallback
		}

		return profileNames;
	}

	// Méthode pour récupérer les PermissionSets via requête SOQL
	private List<String> getPermissionSetMembersByQuery() throws ConnectionException {
		List<String> permissionSetNames = new ArrayList<>();

		try {
			if (!canQueryObject("PermissionSet")) {
				System.out.println("    ⚠ Pas d'accès à l'objet PermissionSet, utilisation du wildcard");
				return permissionSetNames;
			}

			String soqlQuery = "SELECT Name FROM PermissionSet WHERE IsOwnedByProfile = false AND Name != null ORDER BY Name";
			System.out.println("    Exécution de la requête SOQL : " + soqlQuery);

			QueryResult queryResult = partnerConnection.query(soqlQuery);

			if (queryResult.getSize() > 0) {
				SObject[] records = queryResult.getRecords();

				for (SObject record : records) {
					String permSetName = (String) record.getField("Name");
					if (permSetName != null && !permSetName.trim().isEmpty()) {
						permissionSetNames.add(permSetName);
					}
				}

				// Gérer la pagination
				while (!queryResult.isDone()) {
					queryResult = partnerConnection.queryMore(queryResult.getQueryLocator());
					SObject[] moreRecords = queryResult.getRecords();

					for (SObject record : moreRecords) {
						String permSetName = (String) record.getField("Name");
						if (permSetName != null && !permSetName.trim().isEmpty()) {
							permissionSetNames.add(permSetName);
						}
					}
				}

				System.out.println("    → " + permissionSetNames.size() + " permission sets trouvés via SOQL");

			} else {
				System.out.println("    → Aucun permission set trouvé via SOQL");
			}

		} catch (ConnectionException e) {
			System.err.println("    Erreur lors de la requête SOQL pour les permission sets: " + e.getMessage());
		}

		return permissionSetNames;
	}

	// Méthode pour vérifier les permissions avant les requêtes
	private boolean canQueryObject(String objectName) {
		try {
			String testQuery = "SELECT Id FROM " + objectName + " LIMIT 1";
			partnerConnection.query(testQuery);
			return true;
		} catch (ConnectionException e) {
			return false;
		}
	}

	// Méthode améliorée pour collecter les membres avec requêtes SOQL
	private Map<String, List<String>> collectMembersWithSOQL() throws ConnectionException {
		Map<String, List<String>> allMembersCollected = new HashMap<>();

		System.out.println("\n=== COLLECTE DES MEMBRES POUR LES TYPES CLÉS ===");

		for (String typeName : keyMetadataTypes.keySet()) {
			System.out.println("  Traitement de : " + typeName);

			List<String> members = new ArrayList<>();

			try {
				switch (typeName) {
				case "Profile":
					// Utiliser requête SOQL pour les profiles
					members = getProfileMembersByQuery();
					break;

				case "PermissionSet":
					// Utiliser requête SOQL pour les permission sets
					members = getPermissionSetMembersByQuery();
					break;

				case "CustomObject":
					// Pour les objets personnalisés, essayer listMetadata en premier
					List<FileProperties> fileProps = listMetadataMembers(typeName);
					members = fileProps.stream().map(FileProperties::getFullName).collect(Collectors.toList());
					break;

				default:
					// Traitement standard avec listMetadata pour les autres types
					List<FileProperties> standardFileProps = listMetadataMembers(typeName);
					members = standardFileProps.stream().map(FileProperties::getFullName).collect(Collectors.toList());
					break;
				}

			} catch (ConnectionException e) {
				System.err.println("    Erreur lors de la récupération de " + typeName + ": " + e.getMessage());
				// En cas d'erreur, laisser la liste vide pour utiliser le wildcard
			}

			allMembersCollected.put(typeName, members);

			if (!members.isEmpty()) {
				System.out.println("    → " + members.size() + " membres trouvés");
				// Afficher les premiers membres pour debug
				if (members.size() <= 5) {
					System.out.println("    → Membres: " + String.join(", ", members));
				} else {
					System.out.println("    → Premiers membres: " + String.join(", ", members.subList(0, 3)) + "...");
				}
			} else {
				System.out.println("    → Aucun membre trouvé (wildcard sera utilisé si critique)");
			}
		}

		return allMembersCollected;
	}

	private void createMainKeyPackage(Map<String, List<String>> allMembersCollected) throws IOException {
		System.out.println("\n=== CRÉATION DU PACKAGE PRINCIPAL ===");

		Package keyMetadataPackage = new Package();
		keyMetadataPackage.setVersion("58.0");
		List<PackageTypeMembers> keyPackageMembers = new ArrayList<>();
		int keyPackageArtifactCount = 0;

		for (Map.Entry<String, List<String>> entry : keyMetadataTypes.entrySet()) {
			String typeName = entry.getKey();
			List<String> members = allMembersCollected.get(typeName);

			PackageTypeMembers ptm = new PackageTypeMembers();
			ptm.setName(typeName);

			if (members != null && !members.isEmpty()) {
				// Cas normal : des membres ont été trouvés
				ptm.setMembers(members.toArray(new String[0]));
				keyPackageMembers.add(ptm);
				keyPackageArtifactCount += members.size();
				System.out.println("✓ " + typeName + ": " + members.size() + " membres ajoutés");

			} else if (criticalTypes.contains(typeName)) {
				// Cas spécial : type critique sans membres listés -> utiliser wildcard
				ptm.setMembers(new String[] { "*" });
				keyPackageMembers.add(ptm);
				keyPackageArtifactCount += 1;
				System.out.println("⚠ " + typeName + ": Aucun membre listé, wildcard '*' utilisé (type critique)");

			} else {
				// Type non critique sans membres -> exclure
				System.out.println("⊗ " + typeName + ": Exclu (aucun membre et non critique)");
			}
		}

		// Écrire le package principal
		if (!keyPackageMembers.isEmpty()) {
			keyMetadataPackage.setTypes(keyPackageMembers.toArray(new PackageTypeMembers[0]));
			writePackageXml(keyMetadataPackage, "package_main_key_metadata.xml");
			System.out.println("\n✓ Package principal créé: package_main_key_metadata.xml");
			System.out.println("  Total artefacts: " + keyPackageArtifactCount);
			System.out.println("  Types inclus: " + keyPackageMembers.size());
		} else {
			System.out.println("⚠ Aucun type clé à inclure dans le package principal");
		}
	}

	private void createSecondaryPackages(List<DescribeMetadataObject> availableMetadataTypes,
			Map<String, List<String>> allMembersCollected) throws ConnectionException, IOException {

		System.out.println("\n=== CRÉATION DES PACKAGES SECONDAIRES ===");

		List<PackageTypeMembers> currentPackageTypes = new ArrayList<>();
		int currentArtifactCount = 0;
		int packageCounter = 1;

		// Filtrer les types déjà traités dans le package principal
		Set<String> processedKeyTypes = keyMetadataTypes.keySet();
		List<DescribeMetadataObject> remainingTypes = availableMetadataTypes.stream()
				.filter(type -> !processedKeyTypes.contains(type.getXmlName())).collect(Collectors.toList());

		System.out.println("Types restants à traiter: " + remainingTypes.size());

		for (DescribeMetadataObject type : remainingTypes) {
			String typeName = type.getXmlName();

			List<String> members;
			if (allMembersCollected.containsKey(typeName)) {
				members = allMembersCollected.get(typeName);
			} else {
				// Lister les membres pour les types non encore traités
				System.out.println("  Récupération des membres pour : " + typeName + " (non-clé)");
				List<FileProperties> listedMembers = listMetadataMembers(typeName);
				members = listedMembers.stream().map(FileProperties::getFullName).collect(Collectors.toList());
				allMembersCollected.put(typeName, members);
				System.out.println("    Trouvé " + members.size() + " membres pour " + typeName);
			}

			// Si le type a des membres, ou si c'est un type "singleton"
			if (members.isEmpty() && !type.isInFolder() && !type.isMetaFile()) {
				continue; // Skip types without members
			}

			// Calculer le nombre d'artefacts pour ce type
			int typeMembersCount = members.isEmpty() ? 1 : members.size();

			// Vérifier si l'ajout de ce type dépassera la limite du package actuel
			if (currentArtifactCount + typeMembersCount > MAX_ARTIFACTS_PER_PACKAGE) {
				// Écrire le package actuel avant d'en commencer un nouveau
				if (!currentPackageTypes.isEmpty()) {
					writeSecondaryPackage(currentPackageTypes, packageCounter, currentArtifactCount);
					packageCounter++;
				}

				// Réinitialiser pour le nouveau package
				currentPackageTypes = new ArrayList<>();
				currentArtifactCount = 0;
			}

			// Ajouter le type au package actuel
			PackageTypeMembers ptm = new PackageTypeMembers();
			ptm.setName(typeName);
			if (members.isEmpty()) {
				ptm.setMembers(new String[] { "*" });
				currentArtifactCount++;
			} else {
				ptm.setMembers(members.toArray(new String[0]));
				currentArtifactCount += members.size();
			}
			currentPackageTypes.add(ptm);

			System.out.println("  Ajouté: " + typeName + " (" + typeMembersCount + " artefacts)");
		}

		// Écrire le dernier package s'il reste des éléments
		if (!currentPackageTypes.isEmpty()) {
			writeSecondaryPackage(currentPackageTypes, packageCounter, currentArtifactCount);
		}
	}

	private void writeSecondaryPackage(List<PackageTypeMembers> packageTypes, int packageNumber, int artifactCount)
			throws IOException {
		Package pkg = new Package();
		pkg.setTypes(packageTypes.toArray(new PackageTypeMembers[0]));
		pkg.setVersion("58.0");

		String filename = "package_secondary_" + packageNumber + ".xml";
		writePackageXml(pkg, filename);

		System.out.println("✓ Package secondaire créé: " + filename);
		System.out.println("  Artefacts: " + artifactCount + " / " + MAX_ARTIFACTS_PER_PACKAGE);
	}

	private List<FileProperties> listMetadataMembers(String typeName) throws ConnectionException {
		List<FileProperties> members = new ArrayList<>();
		try {
			ListMetadataQuery query = new ListMetadataQuery();
			query.setType(typeName);
			FileProperties[] fileProperties = metadataConnection.listMetadata(new ListMetadataQuery[] { query }, 58.0);
			if (fileProperties != null) {
				members.addAll(Arrays.asList(fileProperties));
			}
		} catch (ConnectionException e) {
			// Certaines métadonnées ne supportent pas `listMetadata` directement
			System.err.println("Avertissement: Erreur lors de la liste des membres pour le type " + typeName + ": "
					+ e.getMessage());
		}
		return members;
	}

	private void writePackageXml(Package pkg, String filename) throws IOException {
		StringBuilder sb = new StringBuilder();
		sb.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n");
		sb.append("<Package xmlns=\"http://soap.sforce.com/2006/04/metadata\">\n");

		for (PackageTypeMembers typeMembers : pkg.getTypes()) {
			sb.append("    <types>\n");
			for (String member : typeMembers.getMembers()) {
				sb.append("        <members>").append(escapeXml(member)).append("</members>\n");
			}
			sb.append("        <name>").append(escapeXml(typeMembers.getName())).append("</name>\n");
			sb.append("    </types>\n");
		}

		sb.append("    <version>").append(pkg.getVersion()).append("</version>\n");
		sb.append("</Package>");

		try (FileWriter writer = new FileWriter(filename)) {
			writer.write(sb.toString());
		}
		System.out.println("Génération du fichier XML: " + filename);
	}

	// Méthode utilitaire pour échapper les caractères XML
	private String escapeXml(String text) {
		if (text == null)
			return "";
		return text.replace("&", "&amp;").replace("<", "&lt;").replace(">", "&gt;").replace("\"", "&quot;").replace("'",
				"&apos;");
	}
}
```
