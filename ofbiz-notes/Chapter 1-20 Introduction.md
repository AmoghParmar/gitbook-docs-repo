# Apache OFBiz - Simple Classroom Notes & Architectural Guide

Welcome to your friendly, step-by-step notes for Apache OFBiz! This guide is written in plain, easy-to-understand English so you can quickly learn how OFBiz works, how requests move through the system, and how to write custom features.

---

## Section 1: Delegator & Database Operations

The **`delegator`** is your primary helper object for reading and writing data in the database.
The `delegator` object is responsible for database operations, record fetching, and full CRUD (Create, Read, Update, Delete) execution in OFBiz.

### Fetching Records
* **`delegator.findOne("<entity-name>", mapObject, useCache)`**:
  * Used when you want to get **a single record** (a single row in the database).
  * You pass the entity name and a Map containing your primary key search conditions.
* **`delegator.find("<entity-name>", condition, ...)`**:
  * Used when you want to get **a list of records**.
  * You pass the entity name and your search conditions using comparison operators (`<`, `>`, `<=`, `>=`, `<>`).

### Modern Entity Query Syntax (`EntityQuery`)
Instead of calling raw delegator methods, OFBiz gives us a clean builder syntax called `EntityQuery`:

```java
// Fetch a single record
GenericValue product = EntityQuery.use(delegator)
    .from("Product")
    .where("productId", productId)
    .queryOne();

// Fetch a list of records
List<GenericValue> productList = EntityQuery.use(delegator)
    .from("Product")
    .where("productTypeId", "FINISHED_GOOD")
    .queryList();
```

---

## Section 2: Configuration, Database Switching & Build Tools

### Switching Databases (`entityengine.xml`)
* `entityengine.xml` is the configuration file where delegators and database connections are defined.
* You use this file when you want to switch OFBiz from the default embedded **Derby** database to relational databases like **MySQL** or **PostgreSQL**.

### Build Tools & File Types
* **`./gradlew build`**: Compiles your Java files into `.class` files and packages them into archives.
* **JAR File (`Java Archive`)**: A compressed `.jar` file containing compiled Java classes and supporting libraries.
* **WAR File (`Web Archive`)**: A compressed `.war` web package deployed on the web server (Apache Tomcat). 
  * Each web app deployed via a WAR file is accessed using its unique **Mount Point URL**.
  * All supporting library `.jar` files are placed on the application's **Classpath**.

---

## Section 3: Classpath vs PATH & System Gradle Commands

### Important Concepts
* **Classpath**: The list of folder locations and `.jar` files loaded into memory when OFBiz runs so Java can find all system classes.
* **PATH (`echo $PATH`)**: The system environment variable pointing to directories on your computer where executable binary commands live.

### Essential Gradle Commands
* **`./gradlew ofbiz`**: Starts the embedded Tomcat web server and mounts all configured web applications.
* **`./gradlew load`**: Initializes database tables, creates indexes, and populates initial setup data (seed and seed-initial data).
* **`./gradlew loadAll`**: Loads all components across framework, core applications, and custom plugins.
* **`./gradlew clean`**: Deletes all compiled `.class` files, build output directories, temporary log files, and embedded Derby database files inside the `runtime/` folder.

---

## Section 4: Logging, Debugging & Cache

### Stack Trace Analysis
When an error occurs, the console or log file displays a **Stack Trace**. Learning to read stack traces helps you immediately locate the exact file name and line number where the issue happened.

### In-Memory Caching
OFBiz stores frequently read database records in **runtime memory cache** so future read requests don't need to hit the database, making the app much faster.

### System Logging (`Log4j2`)
Configured via `log4j2.xml`. It manages system log files:
* `ofbiz.log`: Main system runtime log.
* `error.log`: Logged exceptions and errors.
* `debug.log`: Detailed debugging diagnostics.

### Debug Class vs. `System.out.println`
Never use `System.out.println()` in OFBiz code! Instead, use the `Debug` class (`Debug.java`, configured via `debug.properties`):
* `Debug.logInfo("Informational message", module)`
* `Debug.logWarning("Warning message", module)`
* `Debug.logError(e, "Error message", module)`

---

## Section 5: Directory Architecture

OFBiz organizes its codebase into clear directories:

```
ofbiz/
├── framework/    <-- Core framework engine (Entity Engine, Service Engine, Security, etc.)
├── applications/ <-- Built-in applications (Order, Product, Party, Accounting, Webtools)
├── plugins/      <-- Custom client components (Your custom code goes here!)
└── runtime/      <-- Generated logs (runtime/logs/ofbiz.log), Derby DB files, temp files
```

* **`/framework`**: Contains core system engines. Do not modify files here!
* **`/applications`**: Default business components provided by OFBiz.
* **`/plugins`**: Dedicated folder for client-specific custom plugins. Keeping custom code here prevents framework updates from overwriting your work.
* **`/runtime`**: Holds generated log files, embedded Derby database files, and temporary execution files.

---

## Section 6: Data Readers & Ingestion Rules

Data Readers specify which XML data files get loaded into the database:

1. **`seed`**: Mandatory structural data (such as `WorkEffortType` or `StatusItem`) needed for the system to run.
2. **`seed-initial`**: Baseline data loaded only once during initial setup (such as default admin user `admin` with password `ofbiz`, temporal expressions, and system job data).
3. **`demo`**: Sample data used for testing.
4. **`ext`**: Custom client data additions.

> **Production Rule**: Never run `loadAll` on production servers! Only load specific data readers like `seed` and `seed-initial`.

---

## Section 7: Request Processing & Validation

### Validation Levels
1. **Client-Side Validation**: Performed in the browser form before submitting.
2. **Server-Side Validation**: Performed on the server when Tomcat receives the `HttpServletRequest`. The server reads parameters using `request.getParameter()` and validates the data before passing it to backend services.

---

## Section 8: Events vs. Services Comparison

### What is an Event?
An **Event** handles raw HTTP request and response objects (`HttpServletRequest`, `HttpServletResponse`). It returns a `String` control keyword (like `"success"` or `"error"`) and has direct access to the user's session (`request.getSession()`). Events are used to validate input data before calling services.

### What is a Service?
A **Service** is a session-less block of business logic. It takes an input `Map` and returns an output `Map` (using `ServiceUtil.returnSuccess()` or `ServiceUtil.returnError()`). Services can run synchronously (`runSync`) or asynchronously (`runAsync`) and can be scheduled as background jobs.

### Comparison Table

| Feature | Event | Service |
| :--- | :--- | :--- |
| **Inputs / Outputs** | `HttpServletRequest` & `HttpServletResponse` | Input `Map` in, Output `Map` out |
| **Return Type** | `String` (e.g., `"success"`, `"error"`) | `Map` (Success/Error Map) |
| **Session Access** | Has direct access (`request.getSession()`) | Session-less (No direct session access) |
| **Execution Mode** | Synchronous | Synchronous or Asynchronous (`runAsync`) |
| **Definition Required** | Mapped in `controller.xml` | Defined in `servicedef/services.xml` |
| **Primary Goal** | HTTP handling, request validation | Core business logic & database updates |
| **Background Scheduling** | Cannot be scheduled | Can be scheduled via Job Scheduler |

---

## Section 9: Component Internal Structure (`plugins/example`)

A typical OFBiz component contains the following structure:

* **`ofbiz-component.xml`**: Master component configuration file defining entity paths, service paths, webapp mount points, and data readers.
* **`build.gradle` / `build.xml`**: Component build settings.
* **`config/`**: Contains `*UiLabels.xml` (for internationalization `i18n` and localization `l10n`) and `.properties` files (e.g., `general.properties`).
* **`data/`**: XML data files.
* **`dtd/` & `xsd`**: XML document validation schemas.
* **`entitydef/`**: Entity models (`entitymodel.xml`). Data field types (e.g., `type="id"` mapped to `VARCHAR(40)` in SQL and `String` in Java) are defined in `fieldtypederby.xml` / `fieldtypemysql.xml`.
* **`servicedef/`**: Service definitions (`services.xml`) with parameter modes (`IN`, `OUT`, `IN-OUT`) and implementation engines (`entity-auto`, `groovy`, `java`).
* **`src/`**: Java and Groovy source files.
* **`webapp/WEB-INF/`**: Contains `web.xml` (Tomcat descriptor) and `controller.xml` (routing).
* **`webapp/`**: FreeMarker (`.ftl`) templates, JavaScript, and CSS files.
* **`widget/`**: Screen, Form, Menu XML widget files.

---

## Section 10: Control Flow, Controller Architecture & `handlers-controller.xml`

### Request Execution Sequence
```
Browser Request
   │
   ▼
Tomcat Web Server -> web.xml
   │
   ▼
ControlServlet & RequestHandler
   │
   ▼
controller.xml (Request Maps & View Maps)
   │
   ▼
Event (Input Validation & Filtering)
   │
   ▼
Service (Core Business Logic Execution)
   │
   ▼
View Mapping (Screen Widget Rendering / FTL)
   │
   ▼
HTML Page Returned to Browser
```

* **Modular Controllers**: `controller.xml` can include sub-controllers using `<include location="..."/>` (such as `commonController.xml` or `handlers-controller.xml`).
* **`handlers-controller.xml`**: Dedicated system controller file that registers all system **Event Handlers** (Java, Groovy, Service event handlers) and **View Handlers** (Screen Widget `MacroScreenViewHandler`, FreeMarker FTL view handler). It instructs OFBiz how to execute event scripts and render view templates.

---

## Section 11: Screen Widgets, Form Widgets & Decorator Screens

### Screen Widgets
* **Data Preparation**: Done inside the `<action>` tag section.
  * Static assignments use `<set>` (which automatically infers data types).
  * Complex data preparation uses Groovy scripts.
* **Implicit Context Maps**:
  * **`context`**: General screen context map (keys can be accessed directly without `.get()`).
  * **`parameters`**: Map containing request parameters (`parameters.key`), including global `context-param` values defined in `web.xml`.
* **Decorators**: `<decorator-screen>` and `<decorator-section-include>` assemble layout components (header, footer, navigation bar) from master templates (e.g., `CommonScreens.xml`).

### Form Widgets
Form widgets render HTML forms in 3 main layouts:
1. **`single`**: Used for creating or updating a single record.
2. **`list`**: Multi-row tabular display of records.
3. **`multi`**: Provides a single submit button while allowing input fields across multiple rows.

Key form elements include `<auto-field-service>`, `<auto-field-entity>`, and `<override>` (used to mark non-primary-key fields as required).

---

## Section 12: Groovy DSL & Implicit Framework Objects

Groovy is a simple scripting language used for complex screen data preparation and business logic.

### Automatically Injected Objects:
* `context`: Screen execution context map.
* `parameters`: Request parameter map.
* `request` & `response`: HTTP servlet request/response objects.
* `session`: User session object.
* `userLogin`: GenericValue of the logged-in user.
* `locale`: User locale settings.
* `delegator`: Database delegator instance.
* `dispatcher`: Service dispatcher instance.

---

## Section 13: Service Implementation Engines & Programmatic Calls

Services are defined in `servicedef/services.xml` with parameter modes (`mode="IN"`, `mode="OUT"`, `mode="IN-OUT"`).

### Implementation Engines:
1. **`entity-auto`**: Automated single-entity CRUD without writing Java or Groovy code.
2. **`groovy`**: Used for medium-complexity business logic (loops, conditions, entity queries).
3. **`java`**: Used for complex processing, external API integrations, and performance-critical logic.

### Programmatic Invocation:
* Services are called using `LocalDispatcher` (`runSync` or `runAsync`).
* In Groovy DSL, services can be run directly: `runService('<service-name>', [inputMap])`.

---

## Section 14: FreeMarker (FTL) Templating

FreeMarker (`.ftl`) files generate the final HTML output.

* **Implicit Objects Available**: `parameters`, `context`, `session`, `userLogin`, `delegator`, `dispatcher`.
* **Key Directives**:
  * `<#include "path/to/template.ftl">`: Includes another FTL file.
  * `<#assign var = value>`: Assigns variables or calls static utility methods.
  * `<#list sequence as item>`: Iterates over lists/collections (Hash List).
  * **Null Checks & Defaults**: Null checks (`??`, `!`), default value operators (`var!"default"`).

---

## Section 15: Entity Engine Expressions & View Entities (`<view-entity>`)

### Entity Queries
* **`<entity-one>`**: Fetches a single record by primary key (equivalent to `delegator.findOne`).
* **`<entity-find>` / `<entity-and>`**: Fetches a list of records matching equal conditions.
* **`<entity-condition>`**: Provides flexible filtering using comparison operators (`<`, `>`, `<=`, `>=`, `<>`).

### View Entities (`<view-entity>`)
A **`<view-entity>`** defines a virtual multi-table database join created dynamically at runtime (it does not exist as a physical table in the database).

```xml
<view-entity entity-name="NewCustomerView" package-name="co.hotwax.party">
    <member-entity entity-name="Party" entity-alias="P"/>
    <member-entity entity-name="Person" entity-alias="PER" join-from-alias="P">
        <key-map field-name="partyId"/>
    </member-entity>
    <member-entity entity-name="PartyRole" entity-alias="PR" join-from-alias="P">
        <key-map field-name="partyId"/>
        <entity-condition>
            <econdition field-name="roleTypeId" value="CUSTOMER"/>
        </entity-condition>
    </member-entity>
    
    <member-entity entity-name="PartyContactMech" entity-alias="PCME" join-from-alias="P" join-optional="true">
        <key-map field-name="partyId"/>
    </member-entity>
    <member-entity entity-name="ContactMech" entity-alias="CME" join-from-alias="PCME" join-optional="true">
        <key-map field-name="contactMechId"/>
        <entity-condition>
            <econdition field-name="contactMechTypeId" value="EMAIL_ADDRESS"/>
        </entity-condition>
    </member-entity>
    <member-entity entity-name="TelecomNumber" entity-alias="T" join-from-alias="PCME" join-optional="true">
        <key-map field-name="contactMechId"/>
    </member-entity>
    
    <alias entity-alias="P" field-name="partyId"/>
    <alias entity-alias="PER" field-name="firstName"/>
    <alias entity-alias="PER" field-name="lastName"/>
    <alias entity-alias="CME" field-name="infoString" name="email"/>
    <alias entity-alias="T" field-name="contactNumber" name="phone"/>
</view-entity>
```

---

## Section 16: Event Condition Actions (ECA Triggers)

ECAs act as system triggers:

1. **Entity ECA (EECA)**: Triggers on database entity operations (`create`, `store`, `remove`). Evaluates field conditions (e.g. `statusId == 'APPROVED'`) and executes secondary services automatically.
2. **Service ECA (SECA)**: Triggers before or after a service runs. Evaluates service parameters (e.g. `orderTypeId == 'SALES_ORDER'`) and invokes auxiliary services automatically.

---

## Section 17: Helper, Worker & Utility Classes

OFBiz uses standard Java class design patterns:

* **Helper Classes** (e.g. `PartyHelper.java`): Static domain-specific helper functions.
* **Worker Classes** (e.g. `LoginWorker.java`): Static utility methods for handling servlet requests, sessions, and UI context.
* **Utility Classes** (`UtilMisc.java`, `UtilValidate.java`):
  * `UtilMisc.toMap(...)`: Quickly constructs key-value maps (up to 6 pairs).
  * `UtilValidate.isNotEmpty(...)` / `isEmpty(...)`: Performs null and empty validation checks across strings, collections, and maps.
* **Sequence Generation**: `delegator.getNextSeqId("<entity-name>")` generates sequential unique primary key IDs (+1 sequence increment).

---

## Section 18: Connection Pooling, Database Indexing & Thread Pools

* **Connection Pooling**: Configured in `entityengine.xml` (`pool-size`, `pool-minsize`, `pool-maxsize`) to manage open database connections efficiently.
* **Database Indexing**: Performance relies on properly configured primary keys, foreign key constraints, and database indexes (B+ Tree in InnoDB).
* **Service Thread Pool**: Background job execution and thread allocation managed via `security.properties` and `serviceengine.xml`.

---

## Section 19: Transaction Management (JTA) & `TransactionUtil`

OFBiz handles database transactions using the Java Transaction API (JTA) via `TransactionUtil.java`:

* **`TransactionUtil.begin()`**: Starts a new transaction context.
* **`TransactionUtil.commit()`**: Commits active database changes.
* **`TransactionUtil.rollback()`**: Rolls back database mutations if an error occurs.

---

## Section 20: PerformFind & End-to-End System Summary

### PerformFind Service (`performFind`)
`performFind` is a generalized Webtools search service. It takes an input fields map and entity name, dynamically constructs database query conditions, and returns paginated search result lists.

### Complete End-to-End Request Lifecycle
```
1. User submits Browser Request
2. Tomcat Web Server passes request to ControlServlet
3. RequestHandler checks URI in controller.xml
4. Event validates HTTP parameters (HttpServletRequest)
5. Service Engine executes Business Logic (Java / Groovy)
6. Entity Engine (Delegator & EntityQuery) generates SQL queries
7. TransactionUtil commits updates to Database (MySQL / Derby)
8. Screen Widget & FreeMarker (FTL) render HTML View
9. HTML Page is sent back to User Browser
```
