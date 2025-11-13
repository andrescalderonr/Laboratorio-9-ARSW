### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

### Miembros:

* Jose David Castillo
* Andrés Felipe Calderón

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

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

### Procedimiento:

Entramos al azure portal y buscamos el servicio de Virtual Machines:

![Image](images/part1/VM1.png)

Entramos al servicio, le damos click a Create y luego a Virtual Machine

![](images/part1/CreateVM.png)

Creamos un nuevo resource group de nombre SCALABILITY_LAB

![](images/part1/CreateResource.png)

Colocamos los datos necesarios:

![](images/part1/VMname.png)

![](images/part1/vm2.png)

![](images/part1/vm3.png)

![](images/part1/vmcreated.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

En la sección de conectar esta a de nuestra maquina virtual:

![](images/part1/connect.png)

Usamos el siguiente comando en CMD:

ssh -i "C:\Users\LENOVO\Downloads\Su-llave-ssh-publica.pem" scalability_lab@52.191.3.112

![](images/part1/cmd.png)

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

Corremos este comando para instalar el nvm:

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash

![](images/part1/cmd2.png)

Luego este comando para poder usar el nvm:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

![](images/part1/cmd3.png)

Para instalar Node usamos este comando: nvm install node

![](images/part1/node.png)


4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

Primero instalamos git en la maquina virtual:

![](images/part1/gitcmd.png)

Accedemos a nuestro repo y luego la app, luego instalamos las dependencias

![](images/part1/app.png)

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

Instalamos forever y miramos si el comando funciona

![](images/part1/FiboApp.png)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

![](images/part1/port.png)

![](images/part1/https.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
   Usamos el siguiente comando:

   console.time("fib");
   fetch("http://52.191.3.112:3000/fibonacci/1000000")
   .then(res => res.text())
   .then(data => {
   console.log(data);
   console.timeEnd("fib");
   });

    Y vamos cambiando el final del link con cada uno de los valores que nos pidieron

    * 1000000
   
    ![](images/part1/71.png)

    * 1010000

    ![](images/part1/72.png)

    * 1020000
    
    ![](images/part1/73.png)
    
    * 1030000
   
    ![](images/part1/74.png)

    * 1040000

    ![](images/part1/75.png)

    * 1050000

    ![](images/part1/76.png)

    * 1060000

    ![](images/part1/77.png)

    * 1070000

    ![](images/part1/78.png)

    * 1080000

    ![](images/part1/79.png)

    * 1090000    

    ![](images/part1/710.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

![](images/part1/CPU.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
   
Instalamos newman y entramos a la carpeta postman del repositorio Fibonacci:

![](images/part1/Fibonacci.png)

Editamos el archivo colocando nuestra IP:

![](images/part1/JSON.png)

Corremos el archivo:

![](images/part1/comando.png)

![](images/part1/paso9.png)

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

1[](images/part1/size.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

Paso 7:

"http://52.191.3.112:3000/fibonacci/1000000" = fib: 7217.08203125 ms

"http://52.191.3.112:3000/fibonacci/1010000" = fib: 7604.10107421875 ms

"http://52.191.3.112:3000/fibonacci/1020000" = fib: 7672.085205078125 ms

"http://52.191.3.112:3000/fibonacci/1030000" = fib: 8199.52197265625 ms

"http://52.191.3.112:3000/fibonacci/1040000" = fib: 8307.5009765625 ms

"http://52.191.3.112:3000/fibonacci/1050000" = fib: 8016.992919921875 ms

"http://52.191.3.112:3000/fibonacci/1060000" = fib: 8152.70703125 ms

"http://52.191.3.112:3000/fibonacci/1070000" = fib: 8277.339111328125 ms

"http://52.191.3.112:3000/fibonacci/1080000" = fib: 8463.385986328125 ms

"http://52.191.3.112:3000/fibonacci/1090000" = fib: 8750.07177734375 ms

Paso 8:

![](images/part1/step8.png)

Paso 9:

![](images/part1/step9.png)


12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

Si logramos cumplirlo debido aumentamos el tamaño de la maquina virtual, si logramos ejecutar el comando del paso 9 y que responda como debería, mientras que con el anterior tamaño este no era capaz de dar Ok en la respuesta.

13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

Dejamos la maquina en B1ls

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

Se crean 8 elementos:

* vnet-eastus (Red virtual)
* VERTICAL_SCALABILITY (Máquina virtual)
* vertical_scalability97 (Interfaz de red)
* VERTICAL-SCABILITY-nsg (Grupo de seguridad de red)
* VERTICAL-SCABILITY-ip (Dirección IP pública)
* SCALABILITY_LAB (Grupo de recursos)
* Su-llave-ssh-publica (Clave SSH)
* VERTICAL-SCALABILITY_disk1_fbf3131d8acd48db99254f82d2dcd479 (Disco)

2. ¿Brevemente describa para qué sirve cada recurso?

vnet: Aisla y organiza los recursos de red.

Maquina virtual: Ejecuta el sistema operativo y las aplicaciones.

Interfaz de red: Conecta la maquina virtual a la red virtual.

Grupo de seguridad de red: Controla el trafico en la red.

Ip publica: Permite el acceso remoto.

Grupo de recursos: Agrupa todos los recursos para ser administrados.

Clave ssh: Permite la conección a la maquina virtual.

Disco: Almacena el sistema operativo

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

Al ejecutar el npm FibonacciApp.js, el proceso de Node.js se ejecuta en la misma sesión SSH, debido a esto, al momento de que se cierra la conexión,
la sesión termina y también todos los procesos asociados a esa sesión SSH.

El inbound port rule es lo que permite el tráfico externo hacia nuestro puerto 3000 que es donde corre nuestra aplicación.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.



5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

![](images/part1/CPU.png)

Debido a que para este momento la aplicación no esta optimizada, esta consume una gran cantidad de CPU, alrededor de un 35 a 40 %, en espacial con valores grandes que ademas
tardan mucha más en dar un resultado.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
   
Al inicio hubo fallos debido a que no había suficiente espacio en la máquina virtual para que estas pudieran ejecutarse de forma efectiva, al colocarle más espacio
estas fueron capaces de ejecutarse sin problema alguno.

![](images/part1/step9.png)

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?



8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?



9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?



10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?



11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?




### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

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

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




