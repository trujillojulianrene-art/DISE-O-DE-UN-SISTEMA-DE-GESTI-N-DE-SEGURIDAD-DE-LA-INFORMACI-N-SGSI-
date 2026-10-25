# Laboratorio de validación de controles del SGSI

Entorno: GitHub Codespaces (Ubuntu 24.04), 25 de octubre de 2026. Todas las pruebas se ejecutaron
sobre recursos propios del laboratorio, conforme a la sección 9.6 del informe (consideraciones éticas).

---

## L-01. Copia de seguridad y restauración
**Control:** ISO/IEC 27002 8.13 · **Riesgo:** R02 (pérdida de información contable)

```bash
mkdir -p empresa/contabilidad empresa/clientes
echo 'Factura 001 - Cliente ABC - 1.250.000 COP' > empresa/contabilidad/facturas.txt
echo 'Cliente ABC, NIT 900123456, contacto@abc.com' > empresa/clientes/clientes.csv
sha256sum empresa/*/* | tee hash_original.txt          # huella de los originales
mkdir -p respaldos && tar -czf respaldos/respaldo_$(date +%F).tar.gz empresa
rm -rf empresa                                         # simulación de pérdida (p. ej., ransomware)
tar -xzf respaldos/respaldo_*.tar.gz && sha256sum -c hash_original.txt
```

**Resultado:** respaldo de 306 bytes creado; tras borrar la carpeta, la restauración devolvió
`OK` en los dos archivos: el contenido recuperado es idéntico bit a bit al original. **Superada.**

---

## L-02. Cifrado de información confidencial
**Control:** 8.24 · **Riesgos:** R01, R08

```bash
openssl enc -aes-256-cbc -pbkdf2 -salt -in empresa/clientes/clientes.csv -out clientes.csv.enc
head -c 64 clientes.csv.enc | od -An -c                 # contenido ilegible
openssl enc -d -aes-256-cbc -pbkdf2 -in clientes.csv.enc   # clave incorrecta -> bad decrypt
openssl enc -d -aes-256-cbc -pbkdf2 -in clientes.csv.enc   # clave correcta -> texto original
```

**Resultado:** el archivo cifrado es ilegible (encabezado `Salted__` y bytes aleatorios). Con una
clave incorrecta OpenSSL responde `bad decrypt`; con la clave correcta se recupera el texto original.
La clave no se publica en este repositorio. **Superada.**

---

## L-03. Mínimo privilegio y retiro de accesos
**Controles:** 5.15, 5.18, 8.2 · **Riesgo:** R01

```bash
sudo groupadd contabilidad
sudo useradd -m -s /bin/bash -G contabilidad ana       # contadora
sudo useradd -m -s /bin/bash pedro                     # soporte
sudo mkdir -p /srv/contabilidad && sudo cp empresa/contabilidad/facturas.txt /srv/contabilidad/
sudo chown -R root:contabilidad /srv/contabilidad && sudo chmod -R 770 /srv/contabilidad
sudo su ana   -c 'cat /srv/contabilidad/facturas.txt'  # permitido
sudo su pedro -c 'cat /srv/contabilidad/facturas.txt'  # Permission denied
sudo gpasswd -d ana contabilidad                       # cambio de cargo
sudo su ana   -c 'cat /srv/contabilidad/facturas.txt'  # Permission denied
```

**Resultado:** la carpeta queda con permisos `drwxrwx---`. Ana (grupo contabilidad) lee la factura;
Pedro recibe `Permission denied`. Al retirar a Ana del grupo, pierde el acceso de inmediato. **Superada.**

---

## L-04. Almacenamiento seguro de contraseñas y vencimiento
**Controles:** 5.17, 8.5 · **Riesgo:** R09

```bash
echo 'ana:<clave>' | sudo chpasswd && echo 'pedro:<misma clave>' | sudo chpasswd
sudo grep -E '^(ana|pedro):' /etc/shadow | cut -d: -f1,2
sudo chage -M 90 ana && sudo chage -l ana
```

**Resultado:** con la misma contraseña, los dos usuarios quedan con hashes distintos
(algoritmo yescrypt `$y$` con sal aleatoria), por lo que no se guarda la clave en texto plano ni se
puede detectar su reutilización. La política fija vencimiento a 90 días (expira el 23/01/2027) con
aviso 7 días antes. **Superada.**

---

## L-05. Detección y cierre de un servicio expuesto
**Controles:** 8.20, 8.8 · **Riesgo:** R06

```bash
sudo apt-get install -y nmap
(python3 -m http.server 8080 > /dev/null 2>&1 &)        # servicio inseguro
nmap -p 1-10000 localhost                              # 8080/tcp open
curl -s localhost:8080/empresa/clientes/clientes.csv   # datos expuestos sin autenticación
pkill -f "http.server 8080"                            # tratamiento: cierre del servicio
nmap -p 1-10000 localhost                              # 8080 ya no aparece
curl -s -m 3 localhost:8080 || echo "Servicio cerrado: conexion rechazada"
```

**Resultado:** el escaneo detectó `8080/tcp open` y los datos del cliente se descargaron sin
contraseña (el navegador mostraba el listado completo de archivos). Tras cerrar el servicio, el 8080
desaparece del escaneo y la conexión es rechazada. Los puertos 2000 y 2222 pertenecen a la
infraestructura de Codespaces. **Superada.**

---

## L-06. Autenticación multifactor
**Control:** 8.5 · **Riesgos:** R03, R09

Se verificó en la configuración de la cuenta de GitHub del equipo (*Settings → Password and
authentication*) que la autenticación de dos factores está activa. **Superada.**

---

## Limpieza del entorno
```bash
sudo userdel -r ana; sudo userdel -r pedro; sudo groupdel contabilidad; sudo rm -rf /srv/contabilidad
```
