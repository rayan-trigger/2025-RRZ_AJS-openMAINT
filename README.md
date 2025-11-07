# 📘 README – Dockerisation d’OpenMAINT

## 🚀 Introduction
Ce projet permet de **dockeriser OpenMAINT** (solution de gestion de patrimoine et maintenance basée sur CMDBuild).  
Il utilise :  
- **PostgreSQL + PostGIS** comme base de données  
- **Tomcat 9 + Java 17** pour héberger l’application web OpenMAINT  
- **Docker Compose** pour orchestrer le tout  

---

## 📦 Prérequis

Avant de commencer, assurez-vous d’avoir installé :  
1. **WSL2** (Windows Subsystem for Linux 2)  
   ```powershell
   wsl --install
   ```
   Redémarrez ensuite la machine.  

2. **Docker Desktop for Windows**  
   - Téléchargez : [Docker Desktop](https://www.docker.com/products/docker-desktop/)  
   - Activez l’option **Use WSL 2** pendant l’installation.  

3. Vérifiez que Docker fonctionne :  
   ```powershell
   docker --version
   docker-compose --version
   ```

---

## 📂 Structure du projet

```
openmaint/
├── docker-compose.yml
├── Dockerfile
├── openmaint.war
└── sql/
    ├── cmdbuild-ddl.sql
    ├── cmdbuild-dml.sql
    ├── demo-dataset.sql
```

- `openmaint.war` → l’application OpenMAINT téléchargée depuis [openmaint.org](https://www.openmaint.org/en/download)  
- `sql/` → scripts SQL pour initialiser la base (DDL, DML, dataset démo)  

---

## ⚙️ Configuration

### 1. Dockerfile (Tomcat + OpenMAINT)

```dockerfile
FROM tomcat:9-jdk11

# Variables JVM
ENV CATALINA_OPTS="-Xms512m -Xmx2048m -Dfile.encoding=UTF-8"

# Copier le WAR dans Tomcat
COPY openmaint.war /usr/local/tomcat/webapps/openmaint.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
```

### 2. docker-compose.yml (Base + Application)

```yaml
version: "3.8"

services:
  db:
    image: postgis/postgis:13-3.1
    container_name: openmaint-db
    environment:
      POSTGRES_USER: openmaint
      POSTGRES_PASSWORD: openmaint
      POSTGRES_DB: openmaint
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./sql:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"

  openmaint:
    build: .
    container_name: openmaint-app
    depends_on:
      - db
    ports:
      - "8080:8080"
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_NAME: openmaint
      DB_USER: openmaint
      DB_PASS: openmaint

volumes:
  db_data:
```

---

## ▶️ Lancement

1. Placez-vous dans le dossier du projet :  
   ```powershell
   cd C:\openmaint
   ```

2. Lancez la stack avec :  
   ```powershell
   docker-compose up -d --build
   ```

3. Vérifiez que les conteneurs tournent :  
   ```powershell
   docker ps
   ```

---

## 🌐 Accès à l’application

- Ouvrez votre navigateur sur :  
  👉 [http://localhost:8080/openmaint](http://localhost:8080/openmaint)  

- Identifiants par défaut (si dataset démo chargé) :  
  - **Utilisateur :** `admin`  
  - **Mot de passe :** `admin`  

---

## 🛠️ Commandes utiles

- Arrêter OpenMAINT :  
  ```powershell
  docker-compose down
  ```

- Supprimer aussi les données SQL (⚠️ irréversible) :  
  ```powershell
  docker-compose down -v
  ```

- Voir les logs en direct :  
  ```powershell
  docker-compose logs -f
  ```

---

## 💾 Persistance des données

- Les données PostgreSQL sont stockées dans un volume Docker (`db_data`).  
- Les scripts dans `sql/` sont exécutés uniquement **au premier lancement**.  

---

## ✅ Résumé

Avec ce setup, vous avez :  
- Une base PostgreSQL + PostGIS initialisée avec les scripts OpenMAINT  
- Un Tomcat prêt avec l’application OpenMAINT  
- Une installation reproductible et portable via Docker  
