# AWS Cloud Practitioner - Laboratorio 06

### Objetivo: 
* Interactuar con el servicio AWS CloudFormation
* Identificar características de los siguientes servicios de AWS: VPC, RDS, Secret Manager, EFS, EC2, S3, ALB, Cloud9, EC2 AutoScaling Group, SNS, CloudWatch, CloudFront

---

### A - Actividades Técnicas
<br>

1. Debemos tener una llave Key Pair disponible. De no ser así, acceder al servicio EC2 y luego a la opción "Key Pair". Generar llave RSA y .pem 

2. Acceder al servicio AWS Cloud9 y dar clic en el botón "Create environment"

<img src="images/lab06_01.jpg">
<br>

3. Ingresar "nombre" y "descripción". Dar clic en el botón "Next step"

<img src="images/lab06_02.jpg">
<br>

4. Dejar los valores por defecto. En la sección "Platform" seleccionar "Ubuntu 18.04 TLS". Dar clic en "Next Step"

<img src="images/lab06_03.jpg">
<br>

5. Revisar los valores previamente seleccionados. De estar todo correcto dar clic en el botón "Create enviroment"

6. Esperar unos minutos mientras el ambiente carga.


<img src="images/lab06_04.jpg">
<br>
<img src="images/lab06_05.jpg">
<br>

7. Ejecutar los siguinentes comandos


```bash
#Ubuntu 18.04
sudo apt-get update
git clone https://github.com/jbarreto7991/aws-cloudpractitioner.git
```

<img src="images/lab06_06.jpg">
<br>

8. Acceder al laboratorio 06, carpeta "code". Validar que se cuenta con varios archivos CloudFormation con extensión .yaml. Analizar el contenido de estos archivos.

<img src="images/lab06_07.jpg">
<br>

9. Desplegar cada plantilla CloudFormation ejecutando AWSCLI. Considerar los parámetros a ser ingresados.

    <br>
10. **1_lab06-vpc.yaml**. En la sección "ParameterValue", ingresar el nombre del KeyPair creado en el paso 1. Esta plantilla creará la VPC "192.168.0.0/16" Y 06 Subnets dentro de este CIDR. No deberán existir redes existentes en este rango de IPs. Validar la creación del Stack desde la consola AWS a través del servicio AWS CloudFormation. Analizar el servicio **VPC** y el diagrama de arquitectura facilitado.

```bash
aws cloudformation create-stack --stack-name lab06-vpc --template-body file://~/environment/aws-cloudpractitioner/Lab-06/code/1_lab06-vpc.yaml --parameters ParameterKey=KeyPair,ParameterValue="cloud-practitioner" --capabilities CAPABILITY_IAM
```
    
<img src="images/lab06_08.jpg">
<br>

<img src="images/lab06_09.jpg">
<br>

11. **2_lab06-secretmanager-rds.yaml**. La plantilla solicita el ingreso de 3 parámetros. Se considera el uso de los valores por defecto para estos parámetros. Analizar el servicio **Secret Manager** y **RDS** y el diagrama de arquitectura facilitado. Esta plantilla tomará varios minutos en su despliegue debido al aprovisionamiento de la base de datos.

```bash
aws cloudformation create-stack --stack-name lab06-secretmanager-rds --template-body file://~/environment/aws-cloudpractitioner/Lab-06/code/2_lab06-secretmanager-rds.yaml 
```
   
<br>

12. **3_lab06-efs.yaml**. La plantilla no tiene parámetros. Analizar el servicio de **EFS** y el diagrama de arquitectura facilitado.

```bash
aws cloudformation create-stack --stack-name lab06-efs --template-body file://~/environment/aws-cloudpractitioner/Lab-06/code/3_lab06-efs.yaml 
```

<br>

13. **4_lab06-ec2-s3.yaml**. La plantilla tiene 3 parámetros, 2 son obligatorios: "KeyPair" (Llave creada en el paso 01) y "S3BucketName" (Nombre del Bucket a crear, el nombre debe ser único a nivel mundial. Se recomienda usar el patrón nombre-apellido-cloud-practitioner). Analizar los servicios **EC2** y **S3** y el diagrama de arquitectura facilitado. Esperar unos minutos mientras la aplicación se despliega.

```bash
aws cloudformation create-stack --stack-name lab06-ec2-s3 --template-body file://~/environment/aws-cloudpractitioner/Lab-06/code/4_lab06-ec2-s3.yaml --parameters ParameterKey=KeyPair,ParameterValue="cloud-practitioner" ParameterKey=S3BucketName,ParameterValue="jorge-barreto-cloud-practitioner" --capabilities CAPABILITY_IAM
```

<br>

14. **5_lab06-alb-targetgroup.yaml**. La plantilla no tiene parámetros. Analizar los features del servicio EC2 **ALB (Application Load Balancer), Target Group y Listener** y el diagrama de arquitectura facilitado.

```bash
aws cloudformation create-stack --stack-name lab06-alb-targetgroup --template-body file://~/environment/aws-cloudpractitioner/Lab-06/code/5_lab06-alb-targetgroup.yaml
```

15. A través del endpoint del sitio estático de S3 podemos validar el funcionamiento de nuestra app. Actualmente el servicio S3 (Frontend) apunta a la IP Pública de nuestra instancia EC2 (Backend), el cual apunta a nuestra base de datos RDS. Los comandos a continuación detallan los pasos a seguir para desplegar un balanceador, con el objetivo que nuestro Frontend (S3) no direccione directamente a una instancia EC2 sino a una balanceador. Se deberá validar que el balanceador este desplegado.

```bash

#Ubuntu 18.04
rm -rf /home/ubuntu/environment/aws-cloudpractitioner/
git clone https://github.com/jbarreto7991/aws-cloudpractitioner.git
sudo apt-get install jq -y

#Obteniendo nombre de DNS del Balanceador
REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')
ALB_DNS_NAME=$(aws elbv2 describe-load-balancers --region $REGION | jq -r '.LoadBalancers[] | .DNSName')

#Reemplazando parámetro en archivo de configuración del frontend - Frontend
sed 's+http://$PUBLIC_IP/v1+http://'"$ALB_DNS_NAME"'/v1+g' ~/environment/aws-cloudpractitioner/App/frontend/src/config/axios.js >> ~/environment/aws-cloudpractitioner/App/frontend/src/config/bk_axios.js
rm ~/environment/aws-cloudpractitioner/App/frontend/src/config/axios.js
mv ~/environment/aws-cloudpractitioner/App/frontend/src/config/bk_axios.js ~/environment/aws-cloudpractitioner/App/frontend/src/config/axios.js

#Compilar frontend
cd ~/environment/aws-cloudpractitioner/App/frontend
npm install
npm run build
cd ~/environment/aws-cloudpractitioner/App/frontend/build/

#Enviar actualización a S3
BUCKET=$(aws s3 ls | sort -r | awk 'NR ==1 { print $3 }')
aws s3 sync . s3://$BUCKET

#Registrar instancia EC2 a Target Group del Balanceador
ALB_ARN=$(aws elbv2 describe-load-balancers | jq -r '.LoadBalancers[] | .LoadBalancerArn')
TARGETGROUP_ARN=$(aws elbv2 describe-target-groups --load-balancer-arn $ALB_ARN | jq -r '.TargetGroups[] | .TargetGroupArn')
INSTANCES_ID=$(aws ec2 describe-instances --filters Name=tag-value,Values="EC2 BACKEND" | jq -r '.Reservations[] | .Instances[] | .InstanceId')
aws elbv2 register-targets --target-group-arn $TARGETGROUP_ARN --targets Id=$INSTANCES_ID,Port=80
```
