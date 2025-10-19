# 📚 Apuntes para el Certificado de Ethical Hacker de Cisco

![Project Status](https://img.shields.io/badge/status-completo-green.svg) ![GitHub last commit](https://img.shields.io/github/last-commit/Michel-Macias/Vulnerabilidades-basadas-en-red) ![GitHub repo size](https://img.shields.io/github/repo-size/Michel-Macias/Vulnerabilidades-basadas-en-red) ![License](https://img.shields.io/badge/license-MIT-blue.svg)

## 🎯 ¿Cuál es el objetivo de este repositorio?

Este repositorio nace como una **guía de estudio colaborativa y de acceso libre** para todos los estudiantes que se preparan para el certificado de **Ethical Hacker de Cisco**. El objetivo es desglosar, explicar y practicar los conceptos clave del módulo de "Explotación de Vulnerabilidades Basadas en la Red", uno de los más densos y prácticos del curso.

Aquí encontrarás apuntes teóricos claros y, lo más importante, **laboratorios prácticos** diseñados para que puedas replicar los ataques en tu propio entorno y afianzar los conocimientos.

## 📖 Temario y Contenido

A continuación se muestra la estructura de temas y laboratorios disponibles en esta guía.

| # | Tema | Apuntes Teóricos | Laboratorio Práctico |
|:-:|:---|:---:|:---:|
| 0 | **Montaje del Laboratorio de AD** | [Ver Guía](./1-Redes_Cableadas/00-Guia-Montaje_Lab_AD.md) | *(Guía de configuración)* |
| 1 | **Resolución de Nombres y SMB** | [Ver Teoría](./1-Redes_Cableadas/01-Teoria-Nombres_y_SMB.md) | [Ver Laboratorio](./1-Redes_Cableadas/01.1-Lab-Enumeracion_SMB.md) |
| 2 | **Envenenamiento de Caché DNS** | [Ver Teoría](./1-Redes_Cableadas/02-Teoria-Envenenamiento_DNS.md) | [Ver Laboratorio](./1-Redes_Cableadas/02.1-Lab-Envenenamiento_DNS.md) |
| 3 | **Enumeración de SMTP y SNMP** | [Ver Teoría](./1-Redes_Cableadas/03-Teoria-SMTP_y_SNMP.md) | [Ver Laboratorio](./1-Redes_Cableadas/03.2-Lab-Enumeracion_SMTP_SNMP.md) |
| 4 | **Explotación de FTP** | [Ver Teoría](./1-Redes_Cableadas/04-Teoria-FTP.md) | [Ver Laboratorio](./1-Redes_Cableadas/04.1-Lab-Explotacion_FTP.md) |
| 5 | **Ataques Pass-the-Hash** | [Ver Teoría](./1-Redes_Cableadas/05-Teoria-Pass_The_Hash.md) | [Ver Laboratorio](./1-Redes_Cableadas/05.1-Lab-Pass_The_Hash.md) |
| 6 | **Ataques de Capa 2** | [Ver Teoría](./1-Redes_Cableadas/06-Teoria-Ataques_de_Capa_2.md) | [Ver Laboratorio](./1-Redes_Cableadas/06.1-Lab-SSL_Stripping.md) |
| 7 | **Ataques de Capa 3 (Enrutamiento)** | [Ver Teoría](./1-Redes_Cableadas/07-Teoria-Ataques_de_Capa_3_Enrutamiento.md) | *(Conceptual)* |
| 8 | **Ataques a Kerberos (Kerberoasting)** | [Ver Teoría](./1-Redes_Cableadas/08-Teoria-Ataques_Kerberos_y_LDAP.md) | [Ver Laboratorio](./1-Redes_Cableadas/08.1-Lab-Kerberoasting.md) |
| 9 | **Ataques Criptográficos** | [Ver Teoría](./1-Redes_Cableadas/09-Teoria-Ataques_Criptograficos_y_de_Degradacion.md) | *(Conceptual)* |
| 10 | **Ataques DoS y DDoS** | [Ver Teoría](./1-Redes_Cableadas/10-Teoria-Ataques_DoS_y_DDoS.md) | [Ver Laboratorio](./1-Redes_Cableadas/10.1-Lab-Ataques_DoS.md) |
| 11 | **Omisión de NAC** | [Ver Teoría](./1-Redes_Cableadas/11-Teoria-Omision_de_NAC.md) | [Ver Laboratorio](./1-Redes_Cableadas/11.1-Lab-Omision_de_NAC.md) |


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