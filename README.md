# Trabajo Final - Ingeniería de Software II

## Índice

1. [Equipo](#equipo)
2. [Propósito del Proyecto](#propósito-del-proyecto)
3. [Tecnologías](#tecnologías)
4. [Pipeline](#pipeline)

## Equipo

**Nombre del Proyecto:** ProceedHub ex Hamuk

**Integrantes:**

- Cahuana Nina, Melany Maria Cristina
- Cuela Morales, Andrea Lucia
- Malcoaccha Díaz, Erick Ruben
- Muñoz Curi, Giomar Danny
- Sumire Ramos, Marko Julio
- Taipe Saraza, Christian Daniel
- Zavalaga Orozco, Russell Vanessa

## Propósito del Proyecto

### Objetivo

El objetivo del proyecto es desarrollar una plataforma web accesible y moderna que conecte a las personas con oportunidades de becas, brindando una experiencia atractiva e intuitiva. A través de una interfaz inspiradora, la plataforma busca superar barreras económicas y sociales, promoviendo el empoderamiento educativo y motivando a los usuarios a explorar y aprovechar al máximo estas oportunidades para alcanzar su máximo potencial.

### Arquitectura de Software

El backend sigue una arquitectura basada en Domain-Driven Design (DDD), estructurada en capas con Spring Boot como motor principal. Las capas están organizadas de la siguiente manera:
- **Capa de Aplicación:** Contiene los casos de uso y los servicios RESTful expuestos a los clientes.
- **Capa de Dominio:** Define las entidades del negocio, los agregados, y los servicios del dominio, siguiendo principios de diseño orientado al dominio.
- **Capa de Infraestructura:** Integra la persistencia de datos, servicios externos, y configuraciones de seguridad.
El diseño sigue principios de separación de responsabilidades y extensibilidad para facilitar futuras integraciones. Se emplea un sistema de base de datos relacional (SQL), utilizando PostgreSQL como base de datos principal y Hibernate como framework ORM para gestionar la persistencia. Para pruebas, se incluye H2 como base de datos embebida.

### Funcionalidades principales

#### Gestión de usuarios

- Registro, inicio de sesión y administración de roles.

#### Seguridad

- Autenticación basada en tokens JWT.
- Protección de rutas y servicios REST.

#### Gestión de becas

- Creación, consulta, actualización y eliminación de información de becas.

## Tecnologías

### Lenguajes de Programación

- **Frontend:**
  - JavaScript
  - TypeScript (opcional)
- **Backend:**
  - Java

### Frameworks

- **Frontend:**
  - React
  - Vite
  - TailwindCSS
- **Backend:**
  - Spring Boot
  - Spring Security
  - Spring Data JPA

### Bibliotecas

- **Frontend:**
  - Axios
  - DayJS
  - React Router
  - React Hook Form
  - React Icons
  - JS Cookie
- **Backend:**
  - Lombok
  - JWT
  - JUnit
  - Mockito
  - Spring Boot Starters

## Herramientas de construcción y pruebas

### Frontend

- Vite
- TailwindCSS
- PostCSS
- Autoprefixer
- pnpm
- Selenium

### Backend

- Gradle
- JUnit
- Mockito
- Jacoco
- Thunder Client

## Pipeline

### Configuración del Pipeline

El pipeline automatiza el proceso de construcción, pruebas y análisis estático, además de ejecutar la aplicación.

```groovy
pipeline {
    agent any
    tools {
        jdk 'JAVA'
        gradle 'gradle'
    }
    environment {
        DB_URL = credentials('ProceedHub-db-url')
        DB_USERNAME = credentials('ProceedHub-db-username')
        DB_PASSWORD = credentials('ProceedHub-db-password')
        JWT_EXPIRATION = credentials('ProceedHub-jwt-expiration')
        JWT_TOKEN = credentials('ProceedHub-jwt-token')
        PORT = credentials('ProceedHub-port')
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage("Git Checkout") {
            steps {
                echo "Cloning the develop branch..."
                git branch: 'develop',
                    changelog: false,
                    poll: false,
                    url: 'https://github.com/Natzgun/ProceedHub'
            }
        }
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh './gradlew clean build'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh './gradlew test'
            }
        }
        stage('Copy dependency for Sonarqube') {
            steps {
                echo 'Building the application...'
                sh './gradlew copyDependencies'
            }
        }
        stage("SonarQube Analysis ") {
            steps {
                script {
                    withSonarQubeEnv('sonarqube-proceedhub') {
                        sh """$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=ProceedHub \
                        -Dsonar.projectName=ProceedHub \
                        -Dsonar.sources=. \
                        -Dsonar.tests=src/test/java \
                        -Dsonar.java.libraries=build/libs/*.jar,build/classes/java/main \
                        -Dsonar.java.binaries=. """
                    }
                }
            }
        }
        stage('Run Application') {
            steps {
                echo 'Starting the application...'
                sh 'java -jar build/libs/ProceedHub-0.0.1-SNAPSHOT.jar'
            }
        }
    }
}
```

### Construcción Automática

**Herramienta:** Gradle

```groovy
task build {
    dependsOn 'clean', 'assemble'
    doLast {
        println 'Building ProceedHub application...'
        copy {
            from 'build/libs'
            into 'deploy'
            include '*.jar'
        }
    }
}
```

### Análisis Estático

**Herramienta:** SonarQube

```properties
sonar.projectKey=ProceedHub
sonar.sources=src/main
sonar.tests=src/test
sonar.java.binaries=build/classes
sonar.coverage.exclusions=**/*Config.java
sonar.java.libraries=build/libs/*.jar
```

### Pruebas Unitarias

**Herramientas:** JUnit + Mockito

```java
@Test
void testScholarshipCreation() {
    Scholarship scholarship = new Scholarship();
    scholarship.setTitle("Test Scholarship");
    scholarship.setDescription("Description");
    
    when(scholarshipRepository.save(any(Scholarship.class)))
        .thenReturn(scholarship);
    
    Scholarship saved = scholarshipService.create(scholarship);
    assertEquals("Test Scholarship", saved.getTitle());
}
```

### Pruebas Funcionales

**Herramienta:** Selenium

```java
@Test
public void testLogin() {
    driver.get("http://localhost:3000/login");
    driver.findElement(By.id("email")).sendKeys("test@mail.com");
    driver.findElement(By.id("password")).sendKeys("password");
    driver.findElement(By.id("login-button")).click();
    assertTrue(driver.findElement(By.id("dashboard")).isDisplayed());
}
```

### Pruebas de Seguridad

**Herramienta:** OWASP ZAP

```yaml
auth:
  loginUrl: 'http://localhost:3000/api/auth/login'
  loginRequestData:
    email: '${user}'
    password: '${pass}'
  loginIndicator: "auth-token"

attacks:
  - name: "xss"
  - name: "sql-injection"
  - name: "csrf"
```

### Pruebas de Performance

**Herramienta:** Apache JMeter

```xml
<TestPlan>
  <ThreadGroup>
    <stringProp name="ThreadGroup.num_threads">100</stringProp>
    <stringProp name="ThreadGroup.ramp_time">10</stringProp>
    <HTTPSamplerProxy>
      <stringProp name="HTTPSampler.path">/api/scholarships</stringProp>
      <stringProp name="HTTPSampler.method">GET</stringProp>
    </HTTPSamplerProxy>
  </ThreadGroup>
</TestPlan>
```

### Gestión de Issues

**Issue #30: UpdateScholarshipTest Coverage Analysis**

- **Título:** "Add more tests cases on UpdateScholarshipTest to increase coverage"
- **Etiquetas:** ["unit testing"]
- **Descripción:**
  - Se requiere agregar más casos de prueba para la clase "UpdateScholarshipTest" con el objetivo de aumentar la cobertura de las pruebas unitarias.
- **Código Afectado:** UpdateScholarship.execute()
```
public Scholarship execute(ScholarshipDTO request, String id) {
    Optional<Scholarship> updatedScholarship = scholarshipRepository.findById(id);
    if(updatedScholarship.isEmpty()) {
        throw new IllegalArgumentException("Scholarship with this id does not exist");
    }

    Scholarship beforeScholarship = updatedScholarship.get();
    Scholarship scholarship = Scholarship.builder()
        .id(id)
        .title(request.getTitle() != null && !request.getTitle().isBlank() ? request.getTitle() : beforeScholarship.getTitle())
        .description(request.getDescription() != null && !request.getDescription().isBlank() ? request.getDescription() : beforeScholarship.getDescription())
        .date(request.getDate() != null ? request.getDate() : beforeScholarship.getDate())
        .image(request.getImage() != null && !request.getImage().isBlank() ? request.getImage() : beforeScholarship.getImage())
        .country(request.getCountry() != null && !request.getCountry().isBlank() ? request.getCountry() : beforeScholarship.getCountry())
        .continent(request.getContinent() != null && !request.getContinent().isBlank() ? request.getContinent() : beforeScholarship.getContinent())
        .moreInfo(request.getMoreInfo() != null && !request.getMoreInfo().isBlank() ? request.getMoreInfo() : beforeScholarship.getMoreInfo())
        .requirements(request.getRequirements() != null ? request.getRequirements() : beforeScholarship.getRequirements())
        .build();

    scholarshipRepository.save(scholarship);
    return scholarship;
}
```

**Issue #29: Repository Test Coverage**

- **Título:** "Unit tests for findAll and deleteById from JpaScholarshipRepository"
- **Etiquetas:** ["unit testing"]
- **Descripción:**
  - Se requiere agregar pruebas unitarias para los métodos 'findAll' y 'deleteById' de "JpaScholarshipRepository".
- **Código Afectado:** 
```
@Override
public List<Scholarship> findAll() {
    return repository.findAll()
            .stream()
            .map(ScholarshipMapper::toDomain)
            .toList();
}

@Override
public void deleteById(String id) {
    repository.deleteById(id);
}

```


