# 📚 Apuntes para el Certificado de Ethical Hacker de Cisco

![Project Status](https://img.shields.io/badge/status-en%20progreso-yellow.svg) ![GitHub last commit](https://img.shields.io/github/last-commit/Michel-Macias/Vulnerabilidades-basadas-en-red) ![GitHub repo size](https://img.shields.io/github/repo-size/Michel-Macias/Vulnerabilidades-basadas-en-red) ![License](https://img.shields.io/badge/license-MIT-blue.svg)

## 🎯 ¿Cuál es el objetivo de este repositorio?

Este repositorio nace como una **guía de estudio colaborativa y de acceso libre** para todos los estudiantes que se preparan para el certificado de **Ethical Hacker de Cisco**. El objetivo es desglosar, explicar y practicar los conceptos clave del módulo de "Explotación de Vulnerabilidades Basadas en la Red", uno de los más densos y prácticos del curso.

Aquí encontrarás apuntes teóricos claros y, lo más importante, **laboratorios prácticos** diseñados para que puedas replicar los ataques en tu propio entorno y afianzar los conocimientos.

## 📖 Temario y Contenido Actual

A continuación se muestra la estructura de temas y laboratorios disponibles actualmente. ¡Esta lista crecerá con el tiempo!

| # | Tema | Apuntes Teóricos | Laboratorio Práctico |
|:-:|:---|:---:|:---:|
| 1 | **Resolución de Nombres y SMB** | [Ver Teoría](./docs/01-Teoria-Nombres_y_SMB.md) | [Ver Laboratorio](./docs/01.1-Lab-Enumeracion_SMB.md) |
| 2 | **Envenenamiento de Caché DNS** | [Ver Teoría](./docs/02-Teoria-Envenenamiento_DNS.md) | [Ver Laboratorio](./docs/02.1-Lab-Envenenamiento_DNS.md) |
| 3 | **Enumeración de SMTP y SNMP** | [Ver Teoría](./docs/03-Teoria-SMTP_y_SNMP.md) | [Ver Laboratorio](./docs/03.2-Lab-Enumeracion_SMTP_SNMP.md) |
| 4 | **Explotación de FTP** | [Ver Teoría](./docs/04-Teoria-FTP.md) | [Ver Laboratorio](./docs/04.1-Lab-Explotacion_FTP.md) |
| 5 | **Ataques Pass-the-Hash** | [Ver Teoría](./docs/05-Teoria-Pass_The_Hash.md) | [Ver Laboratorio](./docs/05.1-Lab-Pass_The_Hash.md) |

---
*En construcción: Ataques en Ruta (MitM), SSL Stripping, DoS/DDoS, Omisión de NAC, Salto de VLAN...*

## 🚀 ¿Cómo usar este repositorio?

1.  **Clona el repositorio:** `git clone https://github.com/Michel-Macias/Vulnerabilidades-basadas-en-red.git`
2.  **Lee la teoría:** Cada archivo de teoría está diseñado para ser un resumen conciso y directo del concepto, sus riesgos y sus mitigaciones.
3.  **¡Haz los laboratorios!** La verdadera forma de aprender es practicando. Sigue las guías de los laboratorios en tu propio entorno (por ejemplo, con Kali Linux y una VM de Windows en Virt-Manager).

## 🤝 ¿Quieres contribuir?

¡Toda ayuda es bienvenida! Si encuentras un error, quieres mejorar un apunte o laboratorio, o quieres añadir un tema nuevo, por favor:

1.  Haz un **Fork** del repositorio.
2.  Crea una nueva rama para tus cambios (`git checkout -b mejora-tema-ftp`).
3.  Realiza tus cambios y haz commit (`git commit -m 'Añade más detalles a la explotación de FTP'`).
4.  Haz un **Push** a tu rama (`git push origin mejora-tema-ftp`).
5.  Abre un **Pull Request**.

## 📜 Licencia

Este proyecto está bajo la Licencia MIT. Eres libre de usar, modificar y distribuir el contenido.