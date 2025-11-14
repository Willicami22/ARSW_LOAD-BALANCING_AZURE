### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

### Integrantes
- William Camilo Hernandez Deaza
- Manuel David Robayo Vega

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Creamos la maquina virtual con las caracteristicas pedidas

![Imágen 1](images/part1/Img1.png)
![Imagen2](/images/part1/Img2.png)

2. Desde nuestra maquina local nos conectamos usando el comando

    `ssh scalability_lab@xxx.xxx.xxx.xxx`
![Imagen2](/images/part1/img3.png)

3. Instalamos Node con las especificaciones recomendadas

![Imagen3](/images/part1/img4.png)
4. Clonamos la aplicación en la maquina virtual

![Imagen4](/images/part1/Img5.png)

5. Ejecutamos la aplicación en el servidor para que quede corriendo

![Imagen6](/images/part1/img6.png)

6. Creamos una regla Inbound Rule en Azure

![Imagen7](/images/part1/img7.png)

7. Desde el navegador comprobamos el endpoint: 
[Imagen8](/images/part1/img8.png)

[Imagen9](/images/part1/img9.png)

[Imagen10](/images/part1/img10.png)

[Imagen11](/images/part1/img11.png)
    
[Imagen12](/images/part1/img12.png)

[Imagen13](/images/part1/img13.png)

[Imagen14](/images/part1/img14.png)

[Imagen15](/images/part1/img15.png)

[Imagen16](/images/part1/img16.png)

[Imagen17](/images/part1/img17.png)

8. Verificamos el consumo de la CPU 

![Imágen 18](images/part1/img18.png)

9. Usando la carpeta de PostMan corroboramos el funcionamiento

![Imagen 19](images/part1/img19.png)

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/img20.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

[Imagen21](/images/part1/img21.png)

[Imagen22](/images/part1/img22.png)

[Imagen23](/images/part1/img23.png)

[Imagen24](/images/part1/img24.png)

[Imagen25](/images/part1/img25.png)

[Imagen26](/images/part1/img26.png)

[Imagen27](/images/part1/img27.png)

[Imagen28](/images/part1/img28.png)

[Imagen29](/images/part1/img29.png)

[Imagen30](/images/part1/img30.png)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.



13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

### Respuestas a las preguntas (Parte 1)

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
   - Recursos típicos: `Virtual Machine`, `Network Interface (NIC)`, `OS Disk` (Managed Disk), `Public IP` (si se pidió), `Network Security Group (NSG)` y sus reglas, `Virtual Network (VNet)` + `Subnet`, (opcional) `Storage Account` para boot diagnostics, y (opcional) `Availability Set` o asignación a `Availability Zone`.

2. ¿Brevemente describa para qué sirve cada recurso?
   - `Virtual Machine`: instancia donde corre la app.
   - `NIC`: interfaz de red que conecta la VM a la red.
   - `OS Disk`: disco del sistema operativo y archivos de la VM.
   - `Public IP`: hace accesible la VM desde Internet.
   - `NSG`: controla reglas de entrada/salida (security).
   - `VNet/Subnet`: segmento de red privado donde se despliegan recursos.
   - `Storage Account`: almacena diagnósticos y snapshots.
   - `Availability Set/Zone`: mejora disponibilidad y tolerancia a fallos.

3. ¿Por qué se cae la app al cerrar la conexión SSH y por qué crear una `Inbound port rule`?
   - La app se ejecutó en primer plano dentro de la sesión SSH; al cerrar la sesión el shell envía SIGHUP y termina procesos hijos. Soluciones: usar `forever`, `pm2`, `nohup`, `screen`/`tmux` o un servicio `systemd` para ejecutar la app en background.
   - La `Inbound port rule` en el NSG habilita tráfico entrante (p. ej. puerto 3000). Sin esa regla, Azure bloquea por defecto el acceso desde Internet.

4. Tabla de tiempos (plantilla) e interpretación de por qué tarda tanto la función:
   - Plantilla (completar con datos reales):
     | n | Tiempo (ms) | CPU \% | Memoria (MB) |
     |---:|---:|---:|---:|
     | 1000 |  XX |  YY | ZZ |
     | 10000 |  XX |  YY | ZZ |
     | 100000 |  XX |  YY | ZZ |
   - Interpretación breve: el cálculo de Fibonacci para n grande suele ser CPU-bound y su complejidad depende del algoritmo (recursivo sin memoización = exponencial). Node.js es single-threaded por defecto; cálculos intensivos bloquean el event loop y aumentan tiempos. Además, operaciones con números grandes implican mayor coste de cómputo y memoria.

5. Imagen consumo de CPU (placeholder) e interpretación:
   - Insertar imagen: `images/part1/img18.png` (u otra captura del Monitor de Azure).
   - Interpretación: la CPU se eleva porque la rutina de Fibonacci ejecuta operaciones intensivas en CPU en el proceso principal. En VM pequeñas el porcentaje puede llegar al 100% de los vCPU asignados, provocando cola de peticiones y latencia.

6. Resumen de Postman (placeholder) e interpretación:
   - Insertar imagen resumen de Postman/Newman (archivo de reporte).
   - Interpretación: tiempos por petición aumentan con `n` y con concurrencia; si aparecen fallos suelen ser timeouts o 5xx causados por bloqueo de CPU o por colas que agotan recursos. Documentar códigos de estado, tiempos máximos y cantidad de errores en la tabla de resultados.

7. Diferencia entre tamaños `B2ms` y `B1ls` (no solo specs):
   - `B-series` son máquinas burstables con créditos de CPU: `B2ms` ofrece 2 vCPU y más memoria y mayor baseline de CPU que `B1ls`. `B1ls` es un SKU muy económico y limitado (menos memoria, menor throughput de disco/red).
   - Más allá de specs: la experiencia varía en capacidad sostenida (B2ms mantiene mejor rendimiento sostenido), latencia de I/O y tolerancia al trabajo CPU-bound. `B1ls` puede ser suficiente para cargas muy ligeras pero no para cálculos intensivos.

8. ¿Aumentar el tamaño de la VM es buena solución? ¿Qué pasa con la FibonacciApp al cambiar el tamaño?
   - Subir tamaño ayuda a corto plazo (más vCPU, más RAM → menor latencia). No es solución escalable ni tolerante a fallos.
   - Al cambiar tamaño Azure desasigna/reasigna la VM (detiene/reinicia), por lo que la `FibonacciApp` se interrumpe y debe reiniciarse o gestionarse automáticamente con `forever`/`pm2`/servicio.

9. ¿Qué pasa con la infraestructura al cambiar tamaño? Efectos negativos:
   - Azure dealloca y reprovisiona la VM: downtime temporal, posible cambio de IP si no es estática, reinicio de servicios, coste mayor, y límites de cuotas. Escalar verticalmente no elimina single-point-of-failure y puede ser costoso a mediano/largo plazo.

10. ¿Hubo mejora en consumo de CPU o tiempos? Sí/No y por qué:
   - Generalmente sí: VM más grande reduce porcentaje de uso por núcleo y disminuye latencia por petición. Pero la mejora no siempre es proporcional: si la app es single-threaded, añadir vCPUs no se aprovecha completamente sin paralelismo (worker threads, clustering o múltiples instancias). Además, en contención de I/O o GC la mejora puede ser limitada.

11. Aumentar ejecuciones paralelas a `4`: ¿mejora porcentual?
   - Con una sola VM el throughput puede mejorar hasta saturar CPU → luego los tiempos por petición aumentan y errores pueden aparecer. La mejora porcentual es limitada por la CPU disponible y la naturaleza single-threaded. Con escalamiento horizontal (varias VMs + Load Balancer) la mejora es más lineal y la tasa de éxito suele aumentar porque las peticiones se distribuyen entre instancias.


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




