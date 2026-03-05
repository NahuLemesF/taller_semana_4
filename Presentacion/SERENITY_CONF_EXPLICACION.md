# 📋 Guía del archivo `serenity.conf` en un proyecto Screenplay

## ¿Qué es `serenity.conf`?

El archivo `serenity.conf` es el **archivo de configuración principal** de Serenity BDD. Utiliza el formato **HOCON** (Human-Optimized Config Object Notation) de la librería Typesafe Config. Se ubica en:

```
src/test/resources/serenity.conf
```

Este archivo controla **cómo se ejecutan las pruebas**, **qué navegador usar**, **cómo generar reportes** y **en qué ambiente** correr los tests.

---

## 🔧 Secciones principales

### 1. Configuración del WebDriver

Define qué navegador usar y cómo configurarlo.

```hocon
webdriver {
  base.url = "https://todomvc.com/examples/angular/dist/browser/#/all"  # URL base de la app
  driver = chrome                                                        # Navegador a usar
  capabilities {
    browserName = "chrome"
    acceptInsecureCerts = true              # Aceptar certificados inseguros
    unhandledPromptBehavior = accept        # Manejar alertas automáticamente
    "goog:chromeOptions" {
      args = [
        "test-type",                        # Modo de pruebas
        "ignore-certificate-errors",        # Ignorar errores de certificados
        "--window-size=1000,800",           # Tamaño de ventana
        "--remote-allow-origins=*",         # Permitir orígenes remotos
        "incognito",                        # Modo incógnito
        "disable-infobars",                # Desactivar barra de info
        "disable-gpu",                      # Desactivar GPU (para headless)
        "disable-default-apps",            # Desactivar apps por defecto
        "disable-popup-blocking"           # Desactivar bloqueo de popups
      ]
    }
  }
}
```

**Propiedades clave:**
| Propiedad | Descripción |
|-----------|-------------|
| `base.url` | URL de la aplicación bajo prueba |
| `driver` | Navegador: `chrome`, `firefox`, `edge`, `remote` |
| `capabilities` | Configuración W3C del navegador |
| `"goog:chromeOptions"` | Opciones específicas de Chrome (argumentos de inicio) |

---

### 2. Modo Headless

Controla si el navegador se muestra o corre en segundo plano.

```hocon
headless.mode = true   # true = sin ventana visible | false = ventana visible
```

- **`true`**: El navegador corre en segundo plano (ideal para CI/CD)
- **`false`**: Se abre la ventana del navegador (ideal para depuración)

---

### 3. Configuración de Serenity (Reportes y comportamiento)

```hocon
serenity {
  project.name = "Serenity BDD TodoMVC"        # Nombre que aparece en el reporte
  test.root = "net.serenitybdd.demos.todos"     # Paquete raíz de los tests
  tag.failures = "true"                          # Etiquetar fallos
  linked.tags = "issue"                          # Tags vinculados a issues
  restart.browser.for.each = scenario            # Reiniciar navegador por escenario
  logging = verbose                              # Nivel de logging

  # Reportes
  report.test.durations = true                   # Mostrar duración de cada test
  take.screenshots = AFTER_EACH_STEP             # Cuándo tomar capturas
  store.html = FAILURES                          # Guardar HTML del DOM
}
```

**Opciones de `take.screenshots`:**
| Valor | Descripción |
|-------|-------------|
| `AFTER_EACH_STEP` | Captura después de cada paso (más detallado) |
| `FOR_EACH_ACTION` | Captura en cada acción del usuario |
| `FOR_FAILURES` | Solo captura cuando un test falla |
| `BEFORE_AND_AFTER_EACH_STEP` | Captura antes y después de cada paso |
| `DISABLED` | No tomar capturas |

**Opciones de `restart.browser.for.each`:**
| Valor | Descripción |
|-------|-------------|
| `scenario` | Reinicia el navegador en cada escenario (más aislado) |
| `story` | Reinicia por historia |
| `feature` | Reinicia por feature |
| `never` | Nunca reinicia (más rápido pero menos aislado) |

---

### 4. Página por defecto

```hocon
home.page = "https://todomvc.com/examples/angular/dist/browser/#/all"
```

Define la URL a la que el actor navega cuando usa `Open.browserOn()`.

---

### 5. Ambientes (Environments)

Permite definir configuraciones diferentes según el ambiente de ejecución.

```hocon
environment = "prod,chrome"    # Ambientes activos por defecto

environments {
  local {
    home.page = "http://localhost:8080/angularjs/#/"    # App local
  }
  prod {
    home.page = "https://todomvc.com/examples/angular/dist/browser/#/all"  # Producción
  }
  chrome {
    webdriver {
      driver = chrome
      autodownload = true    # Descarga automática del chromedriver
      capabilities {
        browserName = "chrome"
        # ... configuración de Chrome
      }
    }
  }
  firefox {
    webdriver {
      capabilities {
        browserName = "firefox"
        # ... configuración de Firefox
      }
    }
  }
  edge {
    webdriver {
      capabilities {
        browserName = "MicrosoftEdge"
        # ... configuración de Edge
      }
    }
  }
}
```

**Para activar un ambiente**, se usa la propiedad del sistema `-Denvironment=`:
```bash
./gradlew test -Denvironment=chrome        # Usa Chrome
./gradlew test -Denvironment=firefox       # Usa Firefox
./gradlew test -Denvironment=prod,chrome   # Producción + Chrome
```

---

### 6. Ambientes de ejecución en la nube (Remotos)

Para correr tests en plataformas como BrowserStack, SauceLabs o LambdaTest:

```hocon
environments {
  browserstack {
    webdriver {
      driver = "remote"
      remote.url = "https://${?BROWSERSTACK_USER}:${?BROWSERSTACK_KEY}@hub.browserstack.com/wd/hub"
      capabilities {
        browserName = "Chrome"
        "bstack:options" {
          os = "Windows"
          osVersion = "11"
        }
      }
    }
  }
  saucelabs {
    webdriver {
      driver = "remote"
      remote.url = "https://${?SAUCE_USERNAME}:${?SAUCE_ACCESS_KEY}@ondemand.us-west-1.saucelabs.com:443/wd/hub"
      # ...
    }
  }
}
```

> **Nota:** Se usa `${?VARIABLE}` (con `?`) para que las variables de entorno sean **opcionales** y no causen error si no están definidas.

---

## 📁 Ubicación en el proyecto

```
screenplay-pattern-todomvc/
├── src/
│   └── test/
│       └── resources/
│           ├── serenity.conf          ← ARCHIVO DE CONFIGURACIÓN
│           └── features/              ← Archivos .feature de Cucumber
├── serenity.properties                ← Propiedades adicionales (mínimas)
└── build.gradle                       ← Configuración de Gradle
```

---

## ⚡ Relación con el Screenplay Pattern

En el patrón Screenplay, `serenity.conf` es esencial porque:

1. **Define dónde navega el Actor**: La `base.url` y `home.page` son las URLs que los actores usan en tareas como `Start.withAnEmptyTodoList()`.
2. **Configura el WebDriver**: El actor interactúa con el navegador a través del driver configurado aquí.
3. **Controla las capturas**: Las screenshots que se toman en cada paso del actor se configuran con `take.screenshots`.
4. **Permite múltiples ambientes**: Un mismo test puede correr en Chrome local, Firefox, o en la nube solo cambiando el `-Denvironment`.

---

## 🚀 Comando rápido para ejecutar

```bash
# Correr tests de Screenplay en Chrome y abrir reporte
./gradlew clean test --tests "net.serenitybdd.demos.todos.screenplay.*" -Denvironment=chrome aggregate && open target/site/serenity/index.html
```
