# Standard Operating Procedure: EthioHRI Faliclity Upgrade to verions 1.5

This document provides a comprehensive checklist for deploying and updating components of the EthioHRI OpenMRS platform on a facility server environment.
Adherence to these procedures is critical for maintaining system integrity and ensuring successful deployments.

---

## 1. Prerequisites & Safety Precautions

- **System Access:** Administrative (sudo) privileges are required for all steps.
- **Deployment Assets:** Ensure the following are ready:
  - Updated `ampathform` directory
  - Custom module `ethiohri-reports-1.0.0-SNAPSHOT.omod`,`ethiohrihelper-1.0.0-SNAPSHOT.omod`,`ethiohri-mamba-1.0.3-SNAPSHOT.omod` files
  - Frontend files (`frontend.zip`)
  - Scripts `caregiver_concept_to_person.sql`,
  - Tools `data extraction tools`
  - Concept dictionary `.zip` file
- **System Backup:** Perform a full backup of both the database and the `.OpenMRS` directory before proceeding.

---

## 2. JSON Form Deployment

This procedure updates the Ampath forms used in the system.

### Steps:
#### 1. Navigate to Openmrs working directory
```bash
cd /usr/share/tomcat/tomcat8/.OpenMRS/
```
#### 2. Remove the existing ampathforms directory
```bash
sudo rm -rf configuration/ampathforms
```
#### 3. Copy the new ampathforms directory to Openmrs configuration directory
```bash
sudo cp -R /ampathform/ configuration/
```
> Note: Replace "/ampathform/" with the location of the ampathforms directory from the deployment source
#### 4. Clear the configuration checksums
```bash
sudo rm -f configuration_checksums/ampathforms
```
#### 5. Set correct ownership and permissions
```bash
sudo chown -R tomcat8:tomcat8 /usr/share/tomcat/tomcat8/.OpenMRS
sudo chmod -R 775 /usr/share/tomcat/tomcat8/.OpenMRS
```
> Note: 775 permissions are more secure than 777 and sufficient for Tomcat.
#### 6. Restart the Tomcat service
```bash
sudo systemctl restart tomcat
```

---

## 3. Custom Module Deployment

### 3.1 Preparing Modules for Deployment
#### !!! For Development Team(Ignore by Deployment Team)
#### Custom Modules:
- ethiohri-reports-1.0.0-SNAPSHOT.omod
- ethiohrihelper-1.0.0-SNAPSHOT.omod
- ethiohri-mamba-1.0.3-SNAPSHOT.omod

#### Considerations (Mamba):

- Copy contents of `openmrs-runtime.properties` file (inside mamba project folder) after line 9 tagged with `#Copy from here` into:
  ```
  /usr/share/tomcat/tomcat8/.OpenMRS/openmrs-runtime.properties
  ```
- Ensure these properties match:
```
mambaetl.analysis.db.username=openmrs
mambaetl.analysis.db.password=Pd123\#@!
mambaetl.analysis.db.openmrs_database=openmrs
```
- If issues persist, drop `analytics_db` and restart tomcat:
```bash
mysql -u openmrs -p
DROP DATABASE analytics_db;
sudo systemctl restart tomcat
```

> Note: The above command must be executed for each module
#### For Deployment Team Only
 Copy contents of `openmrs-runtime.properties` file (inside mamba project folder) after line 9 tagged with `#Copy from here` into:
  ```
  /usr/share/tomcat/tomcat8/.OpenMRS/openmrs-runtime.properties
  ```
- Ensure these properties match:
```
mambaetl.analysis.db.username=openmrs
mambaetl.analysis.db.password=Pd123\#@!
mambaetl.analysis.db.openmrs_database=openmrs
```
- If issues persist or updating the existing mamba deployment, drop `analytics_db` 
```bash
mysql -u openmrs -p
DROP DATABASE analytics_db;
```
### 3.2 Deploying Custom Modules
```bash
cd /usr/share/tomcat/tomcat8/.OpenMRS/modules/
sudo rm -f ethiohri-reports-1.0.0-SNAPSHOT.omod ethiohrihelper-1.0.0-SNAPSHOT.omod ethiohri-mamba-1.0.3-SNAPSHOT.omod
sudo cp /path/to/new/ethiohri-reports-1.0.0-SNAPSHOT.omod .
sudo cp /path/to/new/ethiohrihelper-1.0.0-SNAPSHOT.omod .
sudo cp /path/to/new/ethiohri-mamba-1.0.3-SNAPSHOT.omod .
sudo chown -R tomcat8:tomcat8 /usr/share/tomcat/tomcat8/.OpenMRS
sudo chmod -R 775 /usr/share/tomcat/tomcat8/.OpenMRS
sudo systemctl restart tomcat
```

---

## 4. Frontend Application Build

### Steps:
```bash
cd /usr/share/tomcat/tomcat8/.OpenMRS
```
```bash unzip frontend.zip -d .```

---

## 5. Concept Dictionary Import

### Steps:
1. Login to OpenMRS web interface with administrator account.
2. Go to **Administration > Open Concept Lab > Configuration**.
3. Verify or set the subscription details:
    - **Subscribe URL:** `https://api.openconceptlab.org/orgs/ETHIOHRI/collections/ethiohriCollection/01-Aug-2024-15-45`
    - **Token:** `1234567`
4. Click **Save Changes**.
5. Go to the **Open Concept Lab** section.
6. Click **Import from File**.
7. Select the `.zip` file and click **Import**.

---

## 6. Master Database Synchronization

### Steps:
```bash
cd /usr/share/tomcat/tomcat8/.OpenMRS
sudo nano openmrs-runtime.properties
```

- Change the `connection.url` to:
```
connection.url=jdbc:mysql://localhost:3306/openmrs_master?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8&sessionVariables=default_storage_engine%3DInnoDB
```

- Save and exit.

```bash
sudo rm -rf configuration_checksums
sudo systemctl restart tomcat
```

- Wait for application to fully start.
- If needed, follow **Section 5** to import concepts into the master DB.

### Revert Database Connection:
```bash
sudo nano openmrs-runtime.properties
```
- Change database back to `openmrs` (or your original DB name).

```bash
sudo systemctl restart tomcat8
```

### Clean Serialized Objects table:
```bash
mysql -u openmrs -p
use openmrs_master;
delete from serialized_object;
```

---

**End of SOP**

