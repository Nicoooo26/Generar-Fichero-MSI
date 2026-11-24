# Empaquetado de Software (.EXE a .MSI) y Despliegue por GPO en Active Directory

![Windows Server](https://img.shields.io/badge/Windows_Server-Active_Directory-0078D6?style=flat-square&logo=windows)
![GPO](https://img.shields.io/badge/Deployment-Group_Policy_(GPO)-2B579A?style=flat-square)
![MSI Packaging](https://img.shields.io/badge/Packaging-WinINSTALL_LE-4EAA25?style=flat-square)

Documentación técnica de administración de sistemas Windows que cubre el proceso completo de **reempaquetado de aplicaciones** (conversión de un instalador ejecutable `.exe` a paquete Windows Installer `.msi` mediante instantáneas del sistema) y su **distribución centralizada y desatendida** a los equipos de un dominio mediante Directivas de Grupo (GPO).

> 📄 **Versión descargable:** Puedes consultar o descargar este laboratorio en formato PDF desde [`docs/Generar-fichero-MSI.pdf`](./docs/Generar-fichero-MSI.pdf).

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
