# AWS Cloud Practitioner - Laboratorio 02
### Objetivo: 
* Identificar el componente "Instances Status Checks"
* Crear Alarma de Status Check "Instances Status Checks"

---

### A - Actividades Técnicas
<br>

1. **Lanzar** 01 Instancia EC2 Ubuntu 18.04 LTS. La instancia se deberá configurar según la siguiente información:

* "Instance Type": t2.micro
* Generar una "Key Pair" tipo RSA en formato .pem
* "Network Settings": Por defecto (VPC Default) 
* Crear "Security Group" (Inbound Rules) lo detallado a continuación. 

    |Type Port|  Source  |
    |---------|----------|
    |   22    | IP Home  |
  
    
* Configure Storage: Por defecto

<br>

2. En la sección **"Advanced details - User Data"** agregar el siguiente código
<br><br>

```bash
#!/bin/bash
sudo su
cp /etc/netplan/50-cloud-init.yaml /etc/netplan/backup
echo "" > /etc/netplan/50-cloud-init.yaml
sleep 10
reboot
```

<br>

3. Esperar unos minutos mientras la instancia EC2 enciende y los "Status Checks" (System status checks y Instance Status checks) son inicializados. Tiempo promedio de espera: 3 minutos.

<br>

4. Validar que el estado del status check "Instance Status Checks" es "Instance reachability check failed"
<br><br>

<img src="images/lab02_01.jpg">

<br>

---
### B - Crear Alarma de Status Check "Instances Status Checks"

<br>

1. Creación del tópico SNS y subscripción de un email

<img src="images/lab02_02.jpg">
<img src="images/lab02_03.jpg">
<img src="images/lab02_04.jpg">
<img src="images/lab02_05.jpg">
<img src="images/lab02_06.jpg">
<img src="images/lab02_07.jpg">


<br>

2. Creación de alarma "Instances Status Check" en CloudWatch Alarm


<img src="images/lab02_08.jpg">
<img src="images/lab02_09.jpg">
<img src="images/lab02_10.jpg">
<img src="images/lab02_11.jpg">
<img src="images/lab02_12.jpg">
<img src="images/lab02_13.jpg">
<img src="images/lab02_14.jpg">
<img src="images/lab02_15.jpg">
<img src="images/lab02_16.jpg">
<img src="images/lab02_17.jpg">
<img src="images/lab02_18.jpg">
<img src="images/lab02_19.jpg">
<img src="images/lab02_20.jpg">

