# Empaquetado de Software (.EXE a .MSI) y Despliegue por GPO en Active Directory

![Windows Server](https://img.shields.io/badge/Windows_Server-Active_Directory-0078D6?style=flat-square&logo=windows)
![GPO](https://img.shields.io/badge/Deployment-Group_Policy_(GPO)-2B579A?style=flat-square)
![MSI Packaging](https://img.shields.io/badge/Packaging-WinINSTALL_LE-4EAA25?style=flat-square)

Documentación técnica de administración de sistemas Windows que recoge el ciclo completo de **reempaquetado de aplicaciones** (conversión de un instalador ejecutable `.exe` a paquete Windows Installer `.msi` mediante instantáneas del sistema) y su **distribución centralizada y desatendida** a los equipos de un dominio mediante Directivas de Grupo (GPO).

> 📄 **Versión en PDF:** Puedes consultar o descargar la memoria original de este laboratorio en [`docs/Generar-fichero-MSI.pdf`](./docs/Generar-fichero-MSI.pdf).

---

## Arquitectura y Entorno de Laboratorio

| Componente | Especificación / Valor en el Laboratorio |
| :--- | :--- |
| **Estación de Empaquetado** | Windows 11 Tiny (VM limpia sin software residual) |
| **Herramienta de Empaquetado** | Scalable WinINSTALL LE (`Discover` + `WICONSOLE`) |
| **Software Objetivo** | Notepad++ v8.9.8.1 (x64) |
| **Controlador de Dominio** | `WS25DC1.mss.loc` (Dominio: `mss.loc`) |
| **Recurso Compartido Red** | `\\ws25dc1\dato$\MSI\notepad++.msi` (Carpeta oculta) |
| **Unidad Organizativa (OU)** | `Equipos` |
| **Directiva de Grupo (GPO)** | `Install Notepad` |

---

## Estructura del Repositorio

```text
ad-gpo-msi-deployment/
├── README.md                      # Documentación técnica paso a paso
├── docs/
│   └── Generar-fichero-MSI.pdf    # Memoria original del proyecto en PDF
└── img/                           # Capturas de pantalla del laboratorio
```

---

## Fase 1: Preparación del Entorno de Empaquetado

Para evitar que el paquete `.msi` capture cambios irrelevantes de otros programas en segundo plano, el empaquetado se realiza sobre una máquina virtual limpia con **Windows 11 Tiny**.

1. Creamos el directorio de trabajo `C:\Temp\WinInstall` y copiamos en su interior el instalador `WinINSTALL_LE.exe` junto con los archivos de licencia.

![Carpeta WinInstall en C:\Temp](./img/01-carpeta-wininstall.png)

2. Ejecutamos `WinINSTALL_LE.exe` y completamos el asistente de instalación introduciendo los datos de registro requeridos.

![Asistente de registro de Scalable WinINSTALL LE](./img/02-registro-wininstall.png)

3. Tras finalizar la instalación, nos dirigimos a `C:\Program Files (x86)\Scalable\WinINSTALL\Bin` y creamos accesos directos en el escritorio para las dos herramientas principales:
   * **`Discover.exe`**: Asistente para capturar el estado del sistema antes y después de instalar una aplicación.
   * **`WICONSOLE.exe`**: Consola de administración para editar, consolidar y comprimir paquetes `.msi`.

![Accesos directos y directorio Bin de WinINSTALL](./img/03-accesos-directos-bin.png)

4. Copiamos en el escritorio de la máquina el instalador original de **Notepad++** (`npp.8.9.8.1.Installer.x64.exe`).

---

## Fase 2: Captura de Instantáneas (*Before / After*) con Discover

El método de empaquetado por instantáneas (*snapshot*) compara los archivos y claves del registro de Windows antes y después de una instalación para generar un paquete `.msi` equivalente.

1. Ejecutamos **WinINSTALL LE Discover** para iniciar el asistente de creación de paquetes.

![Inicio del asistente WinINSTALL LE Discover](./img/04-discover-wizard.png)

2. Definimos el nombre de la aplicación (`Notepad`) y establecemos la ruta de destino dentro de `C:\Program Files (x86)\Scalable\WinINSTALL\Bin\Packages\Notepad\`.

![Configuración del nombre y ruta de destino en Discover](./img/05-discover-target.png)

3. En **Drive Selection**, seleccionamos la unidad `C:` (`[-c-]`) y la añadimos a *Drives to Scan* para monitorizar todos los cambios realizados en el disco del sistema.

![Selección de la unidad C: a escanear](./img/06-discover-drives.png)

4. Creamos también un acceso directo en el escritorio hacia `C:\Program Files (x86)\Scalable\WinINSTALL\Bin\Packages` para acceder rápidamente a los paquetes generados.

![Creación de acceso directo a la carpeta Packages](./img/07-acceso-directo-packages.png)

5. El asistente realiza la instantánea inicial (**Before Snapshot**). Al completarse, nos indica que ya podemos lanzar el instalador de la aplicación.

![Instantánea Before completada en Discover](./img/08-before-snapshot.png)

6. Seguimos el orden estricto de ejecución:
   1. Ejecutamos `npp.8.9.8.1.Installer.x64.exe` y completamos la instalación de Notepad++.
   2. Volvemos a ejecutar **Discover** para que realice la instantánea final (**After Snapshot**) y calcule las diferencias.

![Orden de ejecución: instalador de Notepad++ y Discover After](./img/09-instalacion-y-after.png)

7. Al finalizar la comparación, Discover muestra la ventana **Conversion Successful**, confirmando que los cambios se han convertido correctamente a formato Windows Installer.

![Ventana de conversión exitosa en Discover](./img/10-conversion-successful.png)

8. Verificamos en `Bin\Packages\Notepad` que se han generado los directorios capturados (`Program Files`, `ProgramData`, `Users`, `Windows`), el fichero `.msi`, el archivo de configuración `.nai` y las entradas de registro `.reg`.

![Archivos generados en la carpeta Packages tras Discover](./img/11-archivos-discover.png)

---

## Fase 3: Consolidación y Compresión del Paquete en WICONSOLE

1. Abrimos **WICONSOLE**, hacemos clic derecho sobre **Windows Installer Packages** > **Import Package** y seleccionamos el fichero `.msi` generado tras el *After*.

![Importación del paquete MSI en WICONSOLE](./img/12-wiconsole-import.png)

2. Al importar el paquete, WICONSOLE visualiza su contenido y crea una carpeta de trabajo con el `.msi` y su archivo índice `.MXI`.

![Vista del paquete importado en WICONSOLE](./img/13-wiconsole-paquete-importado.png)

3. Copiamos dentro de esta carpeta de trabajo todos los directorios y ficheros auxiliares generados durante el proceso de **Discover** (`Program Files`, `ProgramData`, `Users`, `Windows`, `Notepad.nai` y `Notepad.reg`) para asegurarnos de incluir todos los recursos de la aplicación.

![Consolidación de ficheros y carpetas del paquete](./img/14-consolidacion-archivos.png)

4. Hacemos clic derecho sobre el paquete **Notepad** dentro de WICONSOLE y pulsamos en **Compress...** para empaquetar todos los recursos externos dentro de un único archivo `.msi` autocontenido.

![Opción Compress en WICONSOLE](./img/15-wiconsole-compress.png)

5. Como resultado obtenemos el instalador definitivo `notepad++.msi` (~10,6 MB), listo para ser distribuido por red sin depender de carpetas externas.

![Archivo final notepad++.msi generado](./img/16-notepad-msi-final.png)

---

## Fase 4: Validación Local del Paquete

Antes de desplegar el paquete en el dominio, ejecutamos `notepad++.msi` en un equipo cliente Windows limpio. Comprobamos que la instalación finaliza sin errores y que **Notepad++** se ejecuta correctamente.

![Validación de Notepad++ instalado desde el paquete MSI](./img/17-prueba-instalacion-cliente.png)

---

## Fase 5: Despliegue Automatizado mediante GPO en Active Directory

1. En el Controlador de Dominio (`WS25DC1`), abrimos **Group Policy Management** (`gpmc.msc`) y creamos una nueva GPO llamada **`Install Notepad`**.

![Creación de la GPO Install Notepad](./img/18-creacion-gpo.png)

2. Creamos la carpeta local `C:\Datos\MSI`, depositamos el archivo `notepad++.msi` en su interior y compartimos la carpeta `Datos` como recurso oculto de red bajo el nombre **`Dato$`** (con permisos de lectura para los equipos del dominio).

![Carpeta compartida oculta con el fichero notepad++.msi](./img/19-carpeta-compartida-msi.png)

3. Editamos la GPO **`Install Notepad`** y configuramos la instalación de software por equipo:
   * Navegamos a `Computer Configuration > Policies > Software Settings > Software installation`.
   * Añadimos el paquete apuntando siempre a la **ruta de red UNC** (`\\ws25dc1\dato$\MSI\notepad++.msi`) —nunca a la ruta local `C:\`— y seleccionamos el método de despliegue **Assigned**.

![Asignación del paquete MSI en Software Installation](./img/20-gpo-software-installation.png)

4. Habilitamos la espera de red en el arranque para garantizar que el equipo cliente procese la GPO y descargue el `.msi` antes de mostrar el inicio de sesión:
   * Ruta: `Computer Configuration > Policies > Administrative Templates > System > Logon`
   * Directiva: **Always wait for the network at computer startup and logon** → **Enabled**.

![Habilitación de espera de red en el inicio de sesión](./img/21-gpo-wait-network.png)

5. Finalmente, vinculamos la GPO **`Install Notepad`** a la Unidad Organizativa **`Equipos`** para que se aplique automáticamente en el próximo reinicio de las máquinas cliente.

![Vinculación de la GPO Install Notepad a la OU Equipos](./img/22-gpo-vincular-ou.png)

---

## Buenas Prácticas Aplicadas

* **Uso de rutas UNC (`\\servidor\recurso$\...`):** Indispensable en GPOs de instalación de software, ya que los equipos cliente resuelven la ruta a través de la red y no sobre su propio disco local `C:`.
* **Recurso compartido oculto (`Dato$`):** El símbolo `$` al final oculta la carpeta al explorar la red desde el explorador de archivos, evitando accesos accidentales de usuarios finales.
* **Espera de red síncrona:** Activar *Always wait for the network* evita que Windows omita la instalación del `.msi` durante el arranque rápido antes de inicializar la tarjeta de red.

---

## Autor

* **Nicolas Guañuna**
