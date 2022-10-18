# APICURIO

Se basa en la referencia del siguiente enlace:

https://github.com/Apicurio/apicurio-studio/tree/master/distro/openshift

## 1. Instalación de Apicurio

Primero deberá de iniciar sesión al OpenShift

```shell
source ocp4.config
```

```shell
oc login -u ${OCP4_USER} -p ${OCP4_PASSWORD} ${OCP4_MASTER_API}
```

Crear proyecto con el nombre de **apicurio**

```shell
oc new-project apicurio
```

### 1.1. Ejecutar Template para Apicurio Standalone (Opción 01)

Crear el template de **Apicurio Postgres**

```shell
oc -n apicurio apply -f ./apicurio/apicurio-standalone-template.yml
```

Ejecutar el template de la creación de **Apicurio Studio Standalone**

```shell
oc -n apicurio process apicurio-studio-standalone \
    -p UI_ROUTE=${UI_ROUTE} \
    -p API_ROUTE=${API_ROUTE} \
    -p WS_ROUTE=${WS_ROUTE} \
    -p AUTH_ROUTE=${AUTH_ROUTE} \
    -p GENERATED_DB_USER=${DB_USER} \
    -p DB_NAME=${DB_NAME} \
    -p KC_REALM=${KC_REALM} \
    -p GENERATED_KC_USER=${KC_USER} \
    | oc apply -f -
```

### 1.2. Ejecutar Template para Apicurio Postgres (Opción 02)

Crear el template de **Apicurio Postgres**

```shell
oc -n apicurio apply -f ./apicurio/apicurio-postgres-template.yml
```

Ejecutar el template de la creación de **Apicurio Postgres**

```shell
oc -n apicurio process apicurio-postgres \
    -p DB_NAME=${DB_NAME} \
    -p GENERATED_DB_USER=${DB_USER} \
    | oc apply -f -
```

Usar el comando **oc get secret** para recuperar la contraseña de la Base de Datos.

La contraseña de la Base de Datos se almacenan en el secreto **postgresql**. Al ejecutar el siguiente comando, se recupera la el valor de **database-password**. Luego, debido a que el secreto está codificado en Base64, se decodifica.

```shell
export DB_PASS=$(oc -n apicurio get secret postgresql \
    --template='{{index .data "database-password"}}' | base64 -d; echo)
```

Validar nueva variable de entorno

```shell
echo ${DB_PASS}
```

### 1.3. Ejecutar Template para Apicurio Auth (Opción 02)

Crear el template de **Apicurio Auth**

```shell
oc -n apicurio apply -f ./apicurio/apicurio-auth-template.yml
```

Ejecutar el template de la creación de **Apicurio Auth**

```shell
oc -n apicurio process apicurio-auth \
    -p KC_REALM=${KC_REALM} \
    -p GENERATED_KC_USER=${KC_USER} \
    -p AUTH_ROUTE=${AUTH_ROUTE} \
    -p UI_ROUTE=${UI_ROUTE} \
    -p DB_USER=${DB_USER} \
    -p DB_PASS=${DB_PASS} \
    -p DB_NAME=${DB_NAME} \
    -p DB_VENDOR=${DB_VENDOR} \
    | oc apply -f -
```

Usar el comando **oc get secret** para recuperar la contraseña de la Base de Datos.

La contraseña de la Base de Datos se almacenan en el secreto **apicurio-studio-auth**. Al ejecutar el siguiente comando, se recupera la el valor de **keycloak-password**. Luego, debido a que el secreto está codificado en Base64, se decodifica.

```shell
export KC_PASS=$(oc -n apicurio get secret apicurio-studio-auth \
    --template='{{index .data "keycloak-password"}}' | base64 -d; echo)
```

Validar nueva variable de entorno

```shell
echo ${KC_PASS}
```

### 1.4. Ejecutar Template para Apicurio Studio (Opción 02)

Crear el template de **Apicurio Studio**

```shell
oc -n apicurio apply -f ./apicurio/apicurio-template.yml
```

Ejecutar el template de la creación de **Apicurio Studio**

```shell
oc -n apicurio process apicurio-studio \
    -p UI_ROUTE=${UI_ROUTE} \
    -p API_ROUTE=${API_ROUTE} \
    -p WS_ROUTE=${WS_ROUTE} \
    -p AUTH_ROUTE=${AUTH_ROUTE} \
    -p DB_USER=${DB_USER} \
    -p DB_PASS=${DB_PASS} \
    -p DB_NAME=${DB_NAME} \
    -p KC_REALM=${KC_REALM} \
    -p KC_USER=${KC_USER} \
    -p KC_PASS=${KC_PASS} \
    | oc apply -f -
```

### 1.4. Acceso al portal de Apicurio

Para poder acceder al portal del **Apicurio Keycloak Admin Console**, debemos de obtener el usuario y clave de los secretos.

En el panel izquierdo, hacer click en **Workloads → Secrets**. Luego hacer click en el secreto **apicurio-studio-auth**.

![alt text](./resource/apicurio-install-01.png)

Almacenar los valores **keycloak-user** y **keycloak-password** de para inicar sesión **Apicurio Keycloak Admin Console**.

![alt text](./resource/apicurio-install-02.png)

En el panel izquierdo, hacer click en **Networking → Routes**. Luego hacer click en la url del route **apicurio-studio-auth**.

![alt text](./resource/apicurio-install-03.png)

Acceder con las credenciales almacenadas anteriormente.

![alt text](./resource/apicurio-install-04.png)

En el panel izquierdo, hacer click en **Realm Settings**. Luego hacer click en la pestaña **Login** y mantener la siguiente configuración:

- **User registration:** OFF
- **Edit username:** OFF
- **Forgot password:** OFF
- **Remember Me:** OFF
- **Verify email:** OFF
- **Login with email:** ON
- **Require SSL:** external request

![alt text](./resource/apicurio-install-05.png)

En el panel izquierdo, hacer click en **User**. Luego hacer click en el botón **Add user** para poder acceder a la **Apicurio Console**.

![alt text](./resource/apicurio-install-06.png)

Completar el formulario con los siguientes datos:

- **Email:** falero.otiniano.luis@gmail.com
- **First Name:** Luis
- **Last Name:** Falero Otiniano
- **User Enabled:** ON

![alt text](./resource/apicurio-install-07.png)

Hacer click en la pestaña **Credentials** y completar con el siguiente formulario:

- **Password:** redhat
- **Password Confirmation:** redhat
- **Temporary:** OFF

![alt text](./resource/apicurio-install-08.png)

Hacer click en el botón **Set password**

![alt text](./resource/apicurio-install-09.png)

Hacer click en la pestaña **Role Mappings**, seleccionar en **Client Roles** la opción **realm-management** y asignar los siguientes roles: 

- **manage-clients** 
- **manage-users**

![alt text](./resource/apicurio-install-10.png)

En el panel izquierdo, hacer click en **Networking → Routes**. Luego hacer click en la url del route **apicurio-studio-ui**.

![alt text](./resource/apicurio-install-11.png)

Acceder con las credenciales almacenadas anteriormente.

- **Email:** falero.otiniano.luis@gmail.com
- **Password:** redhat

![alt text](./resource/apicurio-install-12.png)


Muestra la primera pantalla de **Apicurio Console**.

![alt text](./resource/apicurio-install-13.png)

## 2. Instalación de Microcks

Crear proyecto con el nombre de **microcks**

```shell
oc new-project microcks
```

En el panel Administrador, hacer click en **Operators → OperatorHub**

![alt text](./resource/microcks-install-01.png)

Hacer click en **Continue**.

![alt text](./resource/microcks-install-02.png)

Hacer click en **Install**.

![alt text](./resource/microcks-install-03.png)

Completar el formulario con los siguientes parámetros y luego hacer click en **Install**.

- **Update channel:** stable
- **Installed Namespace:** microcks
- **Update approval:** Manual

![alt text](./resource/microcks-install-04.png)

Hacher click en **View Operator** para acceder a la página de detalles del operador.

![alt text](./resource/microcks-install-05.png)

Para implementar un recurso personalizado en **Microcks**, ir a la sección **MicrocksInstall** y hacer click en **Create MicrocksInstall**.

![alt text](./resource/microcks-install-06.png)

Actualizar el YAML de acuerdo con el siguiente fragmento.

```yaml
apiVersion: microcks.github.io/v1alpha1
kind: MicrocksInstall
metadata:
  name: microcks
  namespace: microcks
spec:
  name: microcks
  version: 1.6.0
  microcks:
    replicas: 1
  postman:
    replicas: 1
  keycloak:
    install: true
    persistent: true
    volumeSize: 1Gi
  mongodb:
    install: true
    persistent: true
    volumeSize: 2Gi
```

![alt text](./resource/microcks-install-07.png)

Validar que todos los pods estén en estado **Running**

![alt text](./resource/microcks-install-08.png)

Para poder acceder al portal del **Microcks Keycloak Admin Console**, debemos de obtener el usuario y clave de los secretos.

En el panel izquierdo, hacer click en **Workloads → Secrets**. Luego hacer click en el secreto **microcks-keycloak-admin**.

![alt text](./resource/microcks-install-09.png)

Almacenar los valores **username** y **password** de para inicar sesión **Microcks Keycloak Admin Console**.

![alt text](./resource/microcks-install-10.png)

En el panel izquierdo, hacer click en **Networking → Routes**. Luego hacer click en la url del route **microcks-keycloak**.

![alt text](./resource/microcks-install-11.png)

Hacher click en **Administration Console**

![alt text](./resource/microcks-install-12.png)

Acceder con las credenciales almacenadas anteriormente.

![alt text](./resource/microcks-install-13.png)

En el panel izquierdo, hacer click en **User**. Luego hacer click en el botón **Add user** para poder acceder a la **Microcks Console**.

![alt text](./resource/microcks-install-14.png)

Completar el formulario con los siguientes datos:

- **Username:** lfalero
- **Email:** falero.otiniano.luis@gmail.com
- **First Name:** Luis
- **Last Name:** Falero Otiniano
- **User Enabled:** ON

![alt text](./resource/microcks-install-15.png)

Hacer click en la pestaña **Credentials** y completar con el siguiente formulario:

- **Password:** redhat
- **Password Confirmation:** redhat
- **Temporary:** OFF

![alt text](./resource/microcks-install-16.png)

Hacer click en el botón **Set password**

![alt text](./resource/microcks-install-17.png)

Hacer click en la pestaña **Role Mappings**, seleccionar en **Client Roles** la opción **realm-management** y asignar los siguientes roles: 

- **manage-clients** 
- **manage-users**

![alt text](./resource/microcks-install-18.png)

Para tener acceso administrador, hacer click en la pestaña **Role Mappings**, seleccionar en **Client Roles** la opción **microcks-apps** y asignar los siguientes roles: 

- **admin** 
- **manager**

![alt text](./resource/microcks-install-19.png)

En el panel izquierdo, hacer click en **Networking → Routes**. Luego hacer click en la url del route **microcks**.

![alt text](./resource/microcks-install-20.png)

Acceder con las credenciales almacenadas anteriormente.

- **Email:** lfalero
- **Password:** redhat

![alt text](./resource/microcks-install-21.png)

Muestra la primera pantalla de **Microcks Console**.

![alt text](./resource/microcks-install-22.png)

## 3. Integración de Apicurio con Microcks

Hacer click en el botón **Create new API**

![alt text](./resource/apicurio-microcks-01.png)

Completar el formulario con los siguientes datos:

- **Name:**: Echo API
- **Description:** This is a Echo API design doc
- **Type:** Open API 3.0.2
- **Template:** Pet Store Example

![alt text](./resource/apicurio-microcks-02.png)

Para ver el detalle de la documentación, hacer click en **Preview Documentation**

![alt text](./resource/apicurio-microcks-03.png)

Detalle del OpenAPI

![alt text](./resource/apicurio-microcks-04.png)

Acceder al **Microcks Keycloak**, luego hacer click en **Clients** y seleccionar **microcks-serviceaccount**

![alt text](./resource/apicurio-microcks-05.png)

Hacer click en la pestaña **Credentials** y almacenar el nombre **microcks-serviceaccount** y con el **secret**

![alt text](./resource/apicurio-microcks-06.png)

Para poder integrar Apicurio con Microcks, debemos de agregregar algunas variables de entorno en dos **DeploymentConfigs**

## 3.1. Configuración de Apicurio Studio Backend

En el panel izquierdo, hacer click en **Workloads → DeploymentConfigs**. Luego hacer click en el configmap **apicurio-studio-api**.

![alt text](./resource/apicurio-microcks-07.png)

Hacer click en la pestaña **Environment**

![alt text](./resource/apicurio-microcks-08.png)

Agregar los siguientes valores:

- **APICURIO_MICROCKS_API_URL:** https://microcks-microcks.apps.cluster-xdz9q.xdz9q.sandbox1349.opentlc.com/api
- **APICURIO_MICROCKS_CLIENT_ID:** microcks-serviceaccount
- **APICURIO_MICROCKS_CLIENT_SECRET:** ab54d329-e435-41ae-a900-ec6b3fe15c54

![alt text](./resource/apicurio-microcks-09.png)

## 3.2. Configuración de Apicurio Studio Frontend

En el panel izquierdo, hacer click en **Workloads → DeploymentConfigs**. Luego hacer click en el configmap **apicurio-studio-ui**.

![alt text](./resource/apicurio-microcks-10.png)

Hacer click en la pestaña **Environment** y agregar el siguientes valor:

- **APICURIO_UI_FEATURE_MICROCKS:** true

![alt text](./resource/apicurio-microcks-11.png)

Se habilitó la sección de **API Mocking**. Hacer click en **Mock with Microcks**

![alt text](./resource/apicurio-microcks-12.png)

Hacer click en el botón **Mock API**

![alt text](./resource/apicurio-microcks-13.png)

Se realizó la migración satisfactoriamente. Hacer click en el botón **OK**

![alt text](./resource/apicurio-microcks-14.png)

Muestra el detalle de la integración. Hacer click en el botón **View in Microcks**

![alt text](./resource/apicurio-microcks-15.png)

Muestra el nuevo registro de la API en **Microcks**

![alt text](./resource/apicurio-microcks-16.png)

Selecionar una API para validar

![alt text](./resource/apicurio-microcks-17.png)

Ejecutar el Postman con la ruta copiada anteriormente

![alt text](./resource/apicurio-microcks-18.png)