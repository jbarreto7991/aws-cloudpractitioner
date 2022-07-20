# AWS Cloud Practitioner - Laboratorio 04 

### Objetivo: 
* Interactuar con el servicio AWS CloudFormation
* Identificar características de los siguientes servicios de AWS: EC2, S3 y VPC

---

### A - Actividades Técnicas
<br>

1. Debemos tener una llave Key Pair disponible. De no ser así, acceder al servicio EC2 y luego a la opción "Key Pair". Generar llave RSA y .pem 

2. Acceder al servicio AWS CloudFormation y dar clic en el botón "Create Stack"

<img src="images/lab04_01.jpg">
<br>

3. Seleccionar las opciones "Template is ready" y "Upload a template file". Luego, buscar el archivo "lab04_cloudformation_s3_ec2_db.yaml" y subirlo a AWS CloudFormation (El template se encuentra en: https://github.com/jbarreto7991/aws-cloudpractitioner/blob/main/Lab-04/code/lab04_cloudformation_s3_ec2_db.yaml). Dar clic en el botón "Next"  

<img src="images/lab04_02.jpg">
<br>
<br>

4. Ingresar los siguientes datos en la sección "Stack name"
* Stack name: Lab04

<br>

5. Ingresar los siguientes datos en la sección "Parameters". Luego, dar clic en el botón "Next".

* InstancesFamily: Dejar valores por defecto. La instancia EC2 tomará el primer valor de esta lista.
* KeyPair: Seleccionar llave indicada en el paso 1
* S3BucketName: Ingresar un valor único a nivel global. Se recomienda la estructura: nombre-apellido.aws-cloudpractitioner
* SubnetCIDR1: Dejar valor por defecto
* SubnetCIDR2: Dejar valor por defecto
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


