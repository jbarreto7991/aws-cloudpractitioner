# AWS Cloud Practitioner - Laboratorio 04 

### Objetivo: 
* Interactuar con el servicio AWS CloudFormation
* Identificar características de los siguientes servicios de AWS: EC2, EBS, S3 y VPC

---

### A - Actividades Técnicas
<br>

1. Debemos tener una llave Key Pair disponible. De no ser así, acceder al servicio EC2 y luego a la opción "Key Pair". Generar llave RSA y .pem 

2. Acceder al servicio AWS CloudFormation y dar clic en el botón "Create Stack"

<img src="images/lab04_01.jpg">
<br>

3. Seleccionar las opciones "Template is ready" (Sección "Prerequisite - Prepare template") y "Upload a template file" (Sección "Specify template"). Luego, buscar el archivo "lab04_cloudformation_s3_ec2_db.yaml" y subirlo a AWS CloudFormation (El template se encuentra en: https://github.com/jbarreto7991/aws-cloudpractitioner/blob/main/Lab-04/code/lab04_cloudformation_s3_ec2_db.yaml). Dar clic en el botón "Next"  

<img src="images/lab04_02.jpg">
<br>
<br>

4. Ingresar el siguiente dato en la sección "Stack name"
* Stack name: Lab04

<br>

5. Ingresar los siguientes datos en la sección "Parameters". Luego, dar clic en el botón "Next".

* InstancesFamily: Valor por defecto: t2.micro
* KeyPair: Seleccionar KeyPair detallada en el paso 1
* S3BucketName: Ingresar un valor único a nivel global. Se recomienda la estructura: nombre-apellido-aws-cloudpractitioner
* SubnetCIDR1: Valor por defecto
* SubnetCIDR2: Valor por defecto
* VPCCIDR: Dejar valor por defecto. De tener una VPC con CIDR 192.168.0.0/16, modificar valor CIDR por uno donde no exista el solapamiento de redes. Las SubnetCIDR1 y SubnetCIDR2 deben formar parte del VPCCIDR.

<img src="images/lab04_03.jpg">
<br>
<br>

6. No realizar cambios en las secciones "Configure stack options" o "Advanced options". Dar clic en el botón "Next".

<br>

7. En la sección "Capabilities" del último paso dar clic en el checkbox "I acknowledge that AWS CloudFormation might create IAM resources". Luego, dar clic en el botón "Create Stack"

<img src="images/lab04_04.jpg">

8. Esperar unos minutos mientras AWS CloudFormation aprovisiona todos los recursos declarados en la plantilla. 

<img src="images/lab04_05.jpg">
<img src="images/lab04_06.jpg">

9. Analizar las opciones que muestran los servicios AWS CloudFormation, EC2, EBS, VPC y S3.


---
### B - Análisis de los recursos aprovisionados

<br>

Mientras se analizan los siguientes recursos aprovisionados. Diagramar la arquitectura desplegada anteriormente. Se recomienda el uso de "https://app.diagrams.net/"


<img src="images/lab04_12.jpg">
<img src="images/lab04_13.jpg">
<img src="images/lab04_14.jpg">

<br>

### **Networking Services**

Listar los siguientes componentes:

#### **VPC (Virtual Private Cloud)**

1. VPC ID
2. IPv4 CIDR
3. DNS Hostname
4. DNS Resolution


#### **Subnet**

1. Subnet ID
2. IPv4 CIDR
3. Availability Zone
4. Availability Zone ID
5. Available IPv4 addresses
6. Auto-assign public IPv4 address


#### **Internet Gateway**

1. Internet Gateway ID
2. VPC ID

#### **Route Table**

1. Route Table ID
2. Routes
3. Subnet Associations

#### **NACL (Network Access Control List)**

1. NACL ID
2. Inbound Rules
3. Outbound Rules
4. Subnet Association

<br>


### **Compute Services**

Listar los siguientes componentes:

#### **EC2 (Elastic Compute Cloud)**

1. Instances ID
2. Public IPv4 address
3. Private IPv4 address
4. Instance state
5. Public IPv4 hostname
6. Private IPv4 hostname
7. Instance Type
8. Elastic IP address
9. IAM Role
10. Subnet ID
11. AMI ID
12. Key Pair name
13. Subnet ID
14. Availability Zone
15. Network Interface ID

<br>

### **Storage Services**

Listar los siguientes componentes:

#### **EBS (Elastic Block Storage)**

1. Volume ID
2. Type
3. Size
4. Volume State
5. IOPS
6. Throughput
7. Encryption
8. KMS Key Alias
9. Snapshot
10. Availability Zone
11. Attached Instances


#### **S3 (Simple Storage Services)**

1. Bucket Name
2. Objects - Storage Class 
3. Properties - Bucket Versioning
4. Properties - Default Encryption
5. Properties - Static website hosting - Static website hosting
6. Properties - Static website hosting - Bucket website endpint
7. Permissions - Permissions overview - Access
8. Permissions - Block Public Access
9. Permissions - Bucket Policy
10. Permissions - Access control list (ACL)


---

### C - Análisis de la aplicación aprovisionada

<br>

1. Acceder al servicio S3, identificar el bucket creado y acceder al mismo. Ingresar a la sección "Properties" y buscar la sección "Static website hosting". Dar clic en el enlace.

<img src="images/lab04_07.jpg">
<img src="images/lab04_08.jpg">

2. El enlace nos direccionará a una aplicación. Ingresar valores en el campo "Nombre" y en el campo "Descripción". Dar clic en el botón "Create Task".

<img src="images/lab04_09.jpg">

<br>

3. Acceder al servidor (vía SSH o System Manager-Session Manager). Si el método de conexión es SSH, se deberá identificar previamente la IP Pública de la instancia aprovisionada.

```bash
ssh -i PATH/key_pair ubuntu@PUBLIC_IP
ssh -i .\cloud-practitioner.pem ubuntu@44.208.197.7
```
<img src="images/lab04_10.jpg">

<br>

4. Identificamos los siguientes componentes:

#### a. Volúmenes asociados a la instancia

```bash
sudo su
lsblk -fm
df -h
cat /etc/fstab
```

#### b. Versión AWSCLI y listado de recursos

```bash
aws --version
aws s3 ls
aws ec2 describe-instances --region us-east-1
aws cloudformation list-stacks --region us-east-1
```

#### c. Contenido de directorio
```bash
cd /opt/aws-cloudpractitioner/App
```
    
#### d. Base de datos
```bash
mysql
SELECT User, Host FROM mysql.user;
quit

mysql –u admin –p
[password: admin]
show databases;
use test;
show tables;
select * from tasks;
```

<img src="images/lab04_11.jpg">
    
<br>

#### e. Security Groups asociados a la Instancias. ¿A qué se deben estos múltiples registros?

```bash
#Obteniendo la lista de IP desde donde AWS S3 consulta a la instancia EC2. Se considera solo las IP de la región donde se lanza la instancia
REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')
curl https://ip-ranges.amazonaws.com/ip-ranges.json | jq -r '.prefixes[] | select(.region=='\"$REGION\"') | select(.service=="S3") | .ip_prefix' > /opt/tmp_ip_s3_list

#Obteniendo ID VPC
VPC_ID=$(aws ec2 describe-vpcs --filter Name=tag:Name,Values=VPC --query Vpcs[].VpcId --output text --region $REGION)
            
#Obteniendo el ID del Security Group asociado a la instancia
SECURITY_GROUP_NAME=$(curl http://169.254.169.254/latest/meta-data/security-groups)
SECURITY_GROUP_ID=$(aws ec2 describe-security-groups --filter Name=vpc-id,Values=$VPC_ID --region $REGION Name=group-name,Values=$SECURITY_GROUP_NAME | jq -r '.SecurityGroups[] | .GroupId') 
            
#Agregando la lista de IP al Security Group asociado a la instancia
while read IP_S3_LIST; do
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp  --port 80 --cidr $IP_S3_LIST --region $REGION
done < /opt/tmp_ip_s3_list
```

---

### Eliminación de recursos creados
<br>

1. Eliminar contenido del bucket S3 aprovisionado
2. Eliminar Stack de CloudFormation "Lab04"
