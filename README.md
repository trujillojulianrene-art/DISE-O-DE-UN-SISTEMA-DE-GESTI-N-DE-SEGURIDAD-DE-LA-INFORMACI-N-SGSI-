# SGSI TecnoSoluciones S.A.S. – Laboratorio de controles

**Proyecto:** Diseño de un Sistema de Gestión de Seguridad de la Información (SGSI) basado en ISO/IEC 27001:2022
**Autores:** Julián René Trujillo Sabogal y Brandon Andrey Achury Clavijo · **Docente:** Oscar Barragán

TecnoSoluciones S.A.S. es una empresa ficticia (escenario académico). Este repositorio contiene la
validación práctica de los controles propuestos en el SGSI, ejecutada en un entorno propio y aislado
(GitHub Codespaces). No se probó ningún sistema de terceros.

## Contenido

| Archivo / carpeta | Qué es |
|---|---|
| `LABORATORIO.md` | Procedimiento, comandos y resultados de las pruebas L-01 a L-06 |
| `empresa/` | Datos ficticios de la empresa usados en las pruebas |
| `respaldos/` | Copia de seguridad generada en L-01 |
| `hash_original.txt` | Huellas SHA-256 usadas para verificar la restauración |
| `clientes.csv.enc` | Archivo de clientes cifrado con AES-256 en L-02 |

## Resumen de resultados

| Prueba | Control ISO/IEC 27002 | Riesgo | Resultado |
|---|---|---|---|
| L-01 Copia de seguridad y restauración | 8.13 | R02 | Superada |
| L-02 Cifrado de información | 8.24 | R01, R08 | Superada |
| L-03 Mínimo privilegio y retiro de accesos | 5.15, 5.18, 8.2 | R01 | Superada |
| L-04 Almacenamiento de contraseñas y vencimiento | 5.17, 8.5 | R09 | Superada |
| L-05 Detección y cierre de servicio expuesto | 8.20, 8.8 | R06 | Superada |
| L-06 Autenticación multifactor | 8.5 | R03, R09 | Superada |
