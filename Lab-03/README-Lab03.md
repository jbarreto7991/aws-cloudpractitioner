# AWS Cloud Practitioner - Laboratorio 03 (Se incurre en costos)
### Objetivo: 
* Hacer uso de la instancia EC2 Linux con su volumen efímero (EC2 Instance Store).
* Montar un volumen efímero en instancias EC2 usando comandos Linux.
* El tipo de instancia usado en este laboratorio (c5d.large) no se encuentra dentro de la capa gratuita. Tarifa por hora: 0,096 USD.

---

### A - Actividades Técnicas
<br>

1. **Lanzar** 01 Instancia EC2 Ubuntu 18.04 LTS. La instancia se deberá configurar según la siguiente información:
<br>

* "Instance Type": c5d.large
* Generar una "Key Pair" tipo RSA en formato .pem
* "Network Settings": Por defecto (VPC Default) 
* Crear "Security Group" (Inbound Rules) lo detallado a continuación. 

    |Type Port|  Source  |
    |---------|----------|
    |   22    | IP Home  |
  
    
* Configure Storage: Por defecto. Durante el proceso de creación de la instancia EC2 visualizaremos que la instancia cuenta con un volumen root EBS de 8 GiB y un volumen efímero (EC2 Instances Store) de 50 Gb.

<br>

2. Conectarse a la instancia EC2. Ejecutar el siguiente comando. Visualizaremos ambos discos usando los nombres "nvme0n1" (disco root de 8 GiB) y "nvme1n1" (disco efímero de 50 GiB). El disco root nvme1n1 cuenta con su UUID.

```bash
lsblk -fm
```
<img src="images/lab03_01.jpg">

<br>

3. Ejecutar el siguiente comando. Visualizaremos que sólo el disco Root EBS se encuentra montado en la ruta "/". Aún el disco efímero no se encuentra montado en la instancia EC2.

```bash
df -h
```
<img src="images/lab03_02.jpg">
<br>

4. Ejecutar el siguiente comando. Si el resultado es "data", como en este ejemplo, continuar con el siguiente paso.

```bash
sudo file -s /dev/nvme1n1
```
<img src="images/lab03_03.jpg">
<br>


5. Ejecutar el siguiente comando. Procedemos a formatear el disco. El sistema de archivo será "ext4".

```bash
sudo mkfs -t ext4 /dev/nvme1n1
```
<img src="images/lab03_04.jpg">
<br>


6. Ejecutar el siguiente comando. Visualizamos que el disco nvme1n1 cuenta ahora con su código UUID aunque la columna MOUNTPOINT aún no contiene información.

```bash
lsblk -fm
```
<img src="images/lab03_05.jpg">
<br>


7. Ejecutar los siguientes comandos. La columna MOUNTPOINT y "Mount On" ya cuenta con información. 

```bash
mkdir /mount
sudo mount /dev/nvme1n1 /mount
lsblk -fm
```
<img src="images/lab03_06.jpg">
<br>

```bash
df -h
```

<img src="images/lab03_07.jpg">
<br>

8. Ejecutar los siguientes comandos. Generamos un archivo simple y lo guardamos en el disco efímero previamente montado, simulando la persistencia de datos. 

```bash
echo "Test" >> /mount/test
cat /mount/test

```

9. Ejecutar los siguientes comandos. Obtenemos el valor UUID del comanado "lsblk -fm" del disco nvme1n1. Luego, agregamos un registro en el archivo /etc/fstab. Esta última acción permite que el disco sea montado automáticamente.

<img src="images/lab03_08.jpg">
<br>

```bash
nano /etc/fstab
```

```bash
UUID=ee39c1dd-f945-4007-b935-de961f67211e /mount ext4 defaults,nofail 0 2
```


10. Ejecutar los siguientes comandos. Visualizamos en lineas generales que el disco efímero previamente montado, se encuentra correctamente configurado y este contiene un archivo. 

```bash
uptime
cat /mount/test
lsblk -fm
```
<img src="images/lab03_09.jpg">
<br>


11. Desde la consola de AWS procedemos a reiniciar la instancia EC2. Validamos que hay una desconexión SSH desde nuestro terminal. Después de unos segundos, volvemos a ingresar a la instancia EC2. El comando "uptime" indica que la instancia tiene 3 minutos de uso. El reboot no afectó nuestra configuración previa debido a que encontramos el disco efímero montado y podemos leer el contenido del archivo creado.

<img src="images/lab03_10.jpg">

```bash
uptime
cat /mount/test
lsblk -fm
```
<img src="images/lab03_11.jpg">
<br>

12. Realizamos un "stop" y luego un "start" sobre la instancia EC2 desde la consola de AWS y nos logueamos nuevamente a la instancia. Considerar que la IP Pública de la instancia habrá cambiado.


<img src="images/lab03_12.jpg">

<img src="images/lab03_13.jpg">


13. Validamos que la instancia cuenta sólo con el disco root EBS de 8 GiB montado. Al igual que en pasos anteriores, está pendiendo el formateo y asociación el nuevo disco efímero (EC2 Instances Store)

<img src="images/lab03_14.jpg">