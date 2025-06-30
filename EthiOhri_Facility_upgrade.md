# Standard Operating Procedure: EthioHRI Faliclity Upgrade to verions 1.5

This document provides a comprehensive checklist for deploying and updating components of the EthioHRI OpenMRS platform on a facility server environment.
Adherence to these procedures is critical for maintaining system integrity and ensuring successful deployments.

---
Warning Make sure to make backup for the folder .OpenMRS proir to proceeding to the following steps

## 1. Prerequisites & Safety Precautions

- **System Access:** Administrative (sudo) privileges are required for all steps.
- **Deployment Assets:** Ensure the following are ready:
  - Updated `configuration` directory
  -  `modules` files
  - Frontend files (`frontend.zip`)
  - Scripts `caregiver_concept_to_person.sql`,
  - Tools `data extraction tools`
  - Concept dictionary `.zip` file
- **System Backup:** Perform a full backup of both the database and the `.OpenMRS` directory before proceeding.

---

## 2. Configuration Deployment

This procedure updates the Ampath forms used in the system.

### Steps:
#### 1. Navigate to Openmrs working directory
```bash
cd /usr/share/tomcat/tomcat8/.OpenMRS/
```
#### 2. Remove the existing configuration directory
```bash
sudo rm -rf configuration
```
#### 3. Copy the new configuratoin directory to .OpenMRS directory
```bash
sudo cp -R configuration .OpenMRS/
```
> Note: Replace "configuration_checksums" with the location of the configuration directory from the deployment source
#### 4. Clear the configuration checksums
```bash
sudo rm -f configuration_checksums
```
---

## 3. Module Deployment

### 3.1 Preparing Modules for Deployment


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

#### CD folder from the directory `/usr/share/tomcat/tomcat8/.OpenMRS/`
>  Note: Make sure to make a copy of the existing `modules` folder

```bash
cd /usr/share/tomcat/tomcat8/.OpenMRS
```
#### Remove existing folder
```bash
sudo rm -rf modules/
```
##### Copy the new modules into .OpenMRS directory
  ```bash
    cp -rf modules .OpenMRS/
 ```
```bash
sudo chown -R tomcat8:tomcat8 /usr/share/tomcat/tomcat8/.OpenMRS
sudo chmod -R 775 /usr/share/tomcat/tomcat8/.OpenMRS
sudo systemctl restart tomcat
```

---

#### 3.3 Clean Serialized Objects table:
```bash
mysql -u openmrs -p
```
 
```bash
use openmrs;
```
```bash
delete from serialized_object;
```
```bash
exit;
```
#### 3.4 Run the following script to update care gaver record:
  ```bash
  mysql -u openmrs -p openmrs < caregiver_concept_to_person.sql
```
> #### Note Enter password for the mysql openmrs user
  
## 4. Frontend Application Build

### Steps:
```bash
cd /usr/share/tomcat/tomcat8/.OpenMRS
```
```bash 
unzip frontend.zip -d /usr/share/tomcat/tomcat8/.OpenMRS
```

---

## 5. Set correct ownership and permissions
```bash
sudo chown -R tomcat8:tomcat8 /usr/share/tomcat/tomcat8/.OpenMRS
sudo chmod -R 775 /usr/share/tomcat/tomcat8/.OpenMRS
```
> Note: 775 permissions are more secure than 777 and sufficient for Tomcat.
#### 5.1. Restart the Tomcat service
```bash
sudo systemctl restart tomcat
```


#### 6. Concept Dictionary Import

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

**End of SOP**

