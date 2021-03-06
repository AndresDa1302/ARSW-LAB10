### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000   

![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/endpoints%20b1ls.PNG)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/cpu%20vertical.PNG)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

Tiempos de prueba con valores altos 

![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/enpoint%20b2ms.PNG)

Uso medio del CPU 

![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/cpu%20vetical%20op.PNG)

Datos de pruebas postman 

![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/value%201000000%20escalado.PNG)


12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

Cuenta con 12 recursos que son los siguientes: 

![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/recursos.PNG)

2. ¿Brevemente describa para qué sirve cada recurso? 

- La maquina virtual es lo que Azure ofrece para alojar la aplicación sin hardware fisico.
	
	- La clave SSH y el grupo de eseguridad de red mantienen la seguridad en la maquina virtual (integridad, disponibilidad y confidencialidad), junto con reglas que permiten o niegan la entrada o salida de trafico

	- La direccion IP publica nos permite conectarnos remotamente a la maquina virtual
	
	- El disco es el almacenamiento de nuestra maquina virtual

	- La interfaz de red y la vnet nos permite comunicarnos con el internet, recursos de Azure y redes locales de forma segura.


3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
   - Al cerrar la conexión se esta temrinando el proceso que lleva `FibonacciApp.js` y pues este lleva la aplicación por lo que en consecuencia caerá
	
   - El *Inbound port rule* se usa para que se tenga un acceso al servicio publico.


4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
 
 Se menciono lo mal construirdo que esta FibonacciApp, entonces revisando el js vemos que debe realizar muchas recursiones sin un cache que guarde datos anteriores. Se explica mejor con un ejemplo, si buscamos el 10000 entonces las hará y tardará más, por lo que si seguidamente buscamos 11000, tenemos un cache pues seguirá desde 10000 y solo tendrá que hacer 1000 recursiones ahorrando tiempo de ejecución 
 
 Sin Escalamiento
 
 ![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/endpoints%20b1ls.PNG)
 
 Con Escalamiento
 
 ![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/enpoint%20b2ms.PNG)
 

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

 CPU ANTES
 
 ![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/cpu%20vertical.PNG)
 
 CPU DESPUES 
 
 ![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/cpu%20vetical%20op.PNG)

Se puede ver como antes del escalamiento consume más del 60% de la CPU en picos y despues ya este no supera el 20% de consumo pues el tamaño se ha mejorado considerablemente, pasando de 0.5 de RAM a 8 y de vCPU de tener 1 a 2. Por lo que la estrategia de escalamiento se resume en esto claramente

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
 
   	Sin escalamiento
   
	![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/value%20100000.PNG)
   
	Con escalamiento
   
	![](https://github.com/AndresDa1302/ARSW-LAB10/blob/master/images/part1/value%201000000%20escalado.PNG)
   
   
    * Si hubo fallos documentelos y explique.
    
    -Al momento de realizar las pruebas para los postman sin escalar los tiempos de iteracion por pruebas fueron tan altos que no terminaron al usar el valor 1000000 a la tercera iteracion dejo de funcionar correctamente esto se debe a que excedio la memoria disponible y no podia seguir iterando 
    
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
      `B1ls`
         - 1 vCPU
         - 0.5 RAM
         - 2 Discos de datos
         - `B1ls` es mejor para entornos pequeños y de pruebas como servidores web, bases de datos o cualquier otro entorno de desarrollo.
	
      `B2ms`
         - 2 vCPU
         - 8 RAM
         - 4 Discos de datos
         - `B2ms'  parece mas enfocada a entonros de uso medio, ya sea servidores web, conexiones, servidores DNS, pruebas en sistemas operativos con carga media. etc.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

- Es una facil y rapida solución, pues se ve en los cambios de tiempos. Sin embargo una solución que requiere menos costo es mejorar el codigo de FibinacciApp con un cache

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
- LO negativo de estos cambios son los costos mas elevados que implica mantener el tamaño `B2ms`

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
- Los tiempos de respuesta  tienen una diferencia  elevada  por lo que  podemos decir que SI hubo mejora en el consupo de la CPU y tiempos de respuesta.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

-Al aumentar la cantidad de ejecuciones paralelas observamos que el comportamiento del procentaje es mayor debido a quese calculan paralelamete todas las recurrencias permitiendo agilidad y eficiencia.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

- Este fue el resultado cuando se creo el balanceador de carga  y ademas el grupo también. Se puede ver la dirección ip del balanceador que mas adelanta se usara para realizar las peticiones  y ver su correcto funcionamiento.

![](images/part2/createBalancerandGroup.png) 
![](images/part2/IPBalancer.png) 


2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

Despues de realizar todo el proceso como se indica en el laboratorio este fue el resultado al crear las maquinas virtuales (las llaves se agregarán en el repositorio como prueba pero las maquinas estarán apagadas) 

![](images/part2/VMS.png) 

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
``` 

- Estos fueron los siguientes resultados al momento de hacer una petición al balanceador (se puede comprovar que la dirección anteriormente descrita es la misma)

![](images/part2/httpgetBalancer.png)   

![](images/part2/httpgetBalancerFibo.png) 


2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

- Realizando las peticiones simultaneamente como lo indican

![](images/part2/testNewman.png)  

- Tenemos los siguentes resultados 

![](images/part2/resultTest1.png)   

![](images/part2/resultTest2.png)   

##### Comparación peticiones fallidas

Comparandolo con el punto anterior el balanceo de carga horizontal falló mas veces a comparación de la maquina virtualizada teniendo un resultado 3 a 1. 

##### Comparación tiempo de ejecución 

Comparando el tiempo de respuesta de los dos metodos de escalamiento se evidencia que el balanceo de carga obtiene respuesta mucho mas rapida que la otra esto por que cada petición la realiza a una maquina virtual diferente, a diferencia del otro metodo que tiene que calcular todas. (En el metodo vertical se redujo el valor a calcular ya que no se obtenia respuesta). 

##### Comparación costos

Tenemos que por cada maquina virtual ejecutandose hay un costo de  0.027$/hora, ademas por un balanceador de carga 0.022$/hora, ademas por el aumento de memoria para el metodo vertical hay un costo adicional.

Teniendo en cuenta esto es mucho mas costoso realizar el escalamiento horizontal ya que se tienen 3 maquinas y un balanceador de carga.




3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
``` 


A diferencia de las anteriores pruebas la tasa de éxito dismimuyo, (no se sabe en que zona poner la maquina 4) y la siguiente imagen muestra el uso de la CPU para cada maquina al momento de hacer las peticiones GET.
![](images/part2/result4vms.png)  


**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
	- La siguiente imagen tomada de Microsoft permite ver los tipos de balanceadores que existen y estos se diferencian por los protocolos de red que usan
	
	![](images/part2/typeBalancers.png)   
	
	- Azure Load Balancer estará disponible en dos SKU (Unidad de mantenimiento de stocks): Basic y Standard. De forma predeterminada, la SKU estándar se usa al 		crear un clúster de Azure Kubernetes Service(AKS). Con un equilibrador de carga SKU estándar, ofrece más características y funcionalidades, como un tamaño de 		grupo back-end más grande y zonas de disponibilidad. Es importante que comprenda las diferencias entre los equilibradores de carga estándar y básico antes de 		elegir qué utilizar. Después de crear un clúster de AKS, no puede cambiar la SKU del equilibrador de carga para ese clúster. Las API de ambas SKU son similares 	y se invocan a través de la especificación de una SKU.
	
	- Necesitan IP ya que a este se realizaran las peticiones http e internamente este balanceador realizara las peticiones a las maquinas virtuales y poder 		devolver el valor.
	
* ¿Cuál es el propósito del *Backend Pool*? 
	
	- El Backend Pool permite agrupar las maquinas que dispondra el balanceador de carga para poder usarlas. 
	
	![](images/part2/backendPool.png) 
	

* ¿Cuál es el propósito del *Health Probe*? 

	- Al usar reglas de equilibrio de carga con Azure Load Balancer, debe especificar un sondeo de estado para permitir que Load Balancer detecte el estado del 		punto de conexión de back-end. La configuración del sondeo de estado y las respuestas de sondeo determinan qué instancias del grupo de back-end recibirán nuevos 	flujos. Puede usar los sondeos de estado para detectar el error de una aplicación en un punto de conexión de back-end. 
	
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?. 

	- Una regla de equilibrio de carga distribuye el tráfico entrante que se envía a una combinación de dirección IP y puerto seleccionada entre un grupo de 		instancias del grupo de back-end. Solo recibirán nuevo tráfico aquellas instancias de back-end cuyo estado sea correcto según el sondeo de estado. 
	
	- Están disponibles las siguientes opciones: 
	 	*Ninguno (basado en hash): especifica que cualquier máquina virtual puede controlar las solicitudes sucesivas del mismo cliente.*	
		*IP del cliente (afinidad ip de origen de dos tuplas): especifica que las solicitudes sucesivas de la misma dirección IP de cliente serán controladas 			por la misma máquina virtual.*
		*IP y protocolo de cliente (afinidad ip de origen de tres tuplas): especifica que las solicitudes sucesivas de la misma dirección IP de cliente y 			combinación de protocolos serán controladas por la misma máquina virtual.* 
		Esta es bastante importante por que uno define como se van a distribuir las maquinas virtuales por ejemplo la configuración Ip del cliente que con lleva a que solo una maquina estará operando, al momento de hacer una petición a un balanceador lo cual pueden surgir gastos innecesarios por las otras maquinas. 
		
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*? 
	- Es el bloque de creación fundamental de una red privada en Azure. VNet permite muchos tipos de recursos de Azure, como Azure Virtual Machines (máquinas 		virtuales), para comunicarse de forma segura entre usuarios, con Internet y con las redes locales. 
	
	- Es un rango de direcciones lógicas, que se utiliza normalmente cuando se tienen redes demasiado grandes, estos son divididas en redes más pequeñas; estas se 		conocen como subnets
	
	- Los address space son aquellas direcciones de red asignables dentro de una Vnt y los address range son las redes asignables dentro de una subnet.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

	- Una zona de disponibilidad es una oferta de alta disponibilidad que protege sus aplicaciones y datos de los fallos del centro de datos. Las zonas de disponibilidad son ubicaciones físicas únicas dentro de una región de Azure. Cuando una IP es zone-redundant es aquella que replica las peticiones y los datos por medio de las Availability Zone.

* ¿Cuál es el propósito del *Network Security Group*?

	- Puede utilizar un grupo de seguridad de red de Azure para filtrar el tráfico de red hacia y desde los recursos de Azure en una red virtual de Azure. Un 	grupo de seguridad de red contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante hacia, o el tráfico de red saliente desde, varios tipos de recursos de Azure. Para cada regla, puede especificar el origen y el destino, el puerto y el protocolo.

* Informe de newman 1 (Punto 2)
	- Realizado punto anterior
* Presente el Diagrama de Despliegue de la solución.




