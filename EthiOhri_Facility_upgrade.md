# Standard Operating Procedure: EthioHRI Faliclity Upgrade to version 1.5

This document provides a comprehensive checklist for deploying and updating components of the EthioHRI OpenMRS platform on a facility server environment.
Adherence to these procedures is critical for maintaining system integrity and ensuring successful deployments.

---
Warning: Make sure to make backup of the folder .OpenMRS proir to proceeding to the following steps

## 1. Prerequisites & Safety Precautions

- **System Access:** Administrative (sudo) privileges are required for all steps.
- **Deployment Assets:** Get the deployment directory from this link [Deployment](https://cumccolumbia-my.sharepoint.com/:f:/r/personal/atg2151_cumc_columbia_edu/Documents/Ethiohri_Deployment_Tools_June_24_2025/deployment?csf=1&web=1&e=uptcUt)
- **System Backup:** Perform a full backup of both the ethiOhri database and the `/usr/share/tomcat/tomcat8/.OpenMRS` directory before proceeding.

---

## 2. Configuration Deployment

This procedure updates the Ampath forms used in the system.

### Steps:
#### 1. Navigate to Openmrs working directory
```bash
cd /usr/share/tomcat/tomcat8/.OpenMRS/
```
#### 2. Remove existing configuration, modules and frontend directories
```bash
sudo rm -rf configuration modules frontend
```
#### 3. Clear configuration checksums
```bash
sudo rm -rf configuration_checksums
```
#### 4. Copy the new configuratoin, modules and frontend directories to `/usr/share/tomcat/tomcat8/.OpenMRS/`
```bash
sudo cp -R configuration /usr/share/tomcat/tomcat8/.OpenMRS/
```
```bash
sudo cp -R modules /usr/share/tomcat/tomcat8/.OpenMRS/
```
```bash
unzip frontend.zip -d /usr/share/tomcat/tomcat8/.OpenMRS
```
> Note: Run the above commands from the deployment directory
---
#### 5. Set correct ownership and permissions
```bash
sudo chown -R tomcat8:tomcat8 /usr/share/tomcat/tomcat8/.OpenMRS
```
```bash
sudo chmod -R 775 /usr/share/tomcat/tomcat8/.OpenMRS
```
> Note: 775 permissions are more secure than 777 and sufficient for Tomcat.

#### 6. Update openmrs properties for Mamba

- Open `/usr/share/tomcat/tomcat8/.OpenMRS/openmrs-runtime.properties` via an editor e.g nano and add this script at the bottom:

```
#Copy from here
mambaetl.analysis.db.username=openmrs
mambaetl.analysis.incremental_mode=1
mambaetl.analysis.columns=50
mambaetl.analysis.db.etl_database=analytics_db
mambaetl.analysis.db.password=<<password_placeholder>>
mambaetl.analysis.etl_interval=300
mambaetl.analysis.locale=en
mambaetl.analysis.db.url=jdbc\:mysql\://localhost\:3306/analytics_db?autoReconnect\=true&useSSL\=false&allowMultiQueries\=true
mambaetl.analysis.db.openmrs_database=openmrs
mambaetl.analysis.automated_flattening=0
```
- Ensure `mambaetl.analysis.db.username=openmrs` matches with the mysql openmrs user used.
- Ensure to replace `<<password_placeholder>>` with the password for mysql openmrs user you set.
- Ensure `mambaetl.analysis.db.openmrs_database=openmrs` matches with the correct ethiOhri database name used.

- If issues persist, drop `analytics_db` and restart tomcat:
```bash
mysql -u openmrs -p
DROP DATABASE analytics_db;
sudo systemctl restart tomcat
```
---

#### 7. Clean Serialized Objects table:
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
#### 8. Run the following script from deployment directory to update care gaver record:
  ```bash
  mysql -u openmrs -p openmrs < caregiver_concept_to_person.sql
```
> Note: Enter password for the mysql openmrs user when prompted.


#### 9. Concept Dictionary Import

### Steps:
1. Login to OpenMRS web interface with administrator account.
2. Go to **Administration > Open Concept Lab > Configuration**.
3. Verify or set the subscription details:
    - **Subscribe URL:** `https://api.openconceptlab.org/orgs/ETHIOHRI/collections/ethiohriCollection/01-Aug-2024-15-45`
    - **Token:** `1234567`
4. Click **Save Changes**.
5. Go to the **Open Concept Lab** section.
6. Click **Import from File**.
7. Select the first `.zip` file from concepts folder in the deployment directory and click **Import**. Repeat this for all `.zip` files.
---

#### 10. Restart the Tomcat service
```bash
sudo systemctl restart tomcat
```

**End of SOP**

