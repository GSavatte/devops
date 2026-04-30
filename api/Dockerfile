# --- Compilation ---
FROM maven:3.8-eclipse-temurin-11 AS build
WORKDIR /app

# Copie du code source
COPY . .

# Mise en palce des bonnes perms et execution de la commande données dans le readme
RUN chmod +x mvnw
RUN ./mvnw clean package -Dquarkus.package.type=uber-jar -DskipTests

# --- Exécution ---
# Utilisation d'une image java tres legere
FROM eclipse-temurin:25.0.2_10-jre-alpine
WORKDIR /app

# On récupère l'uber-jar généré à l'étape 1
COPY --from=build /app/target/*-runner.jar app.jar

# On expose le port 8080
EXPOSE 8080

# On lance l'application avec la commande du readme
ENTRYPOINT ["java", "-jar", "app.jar"]