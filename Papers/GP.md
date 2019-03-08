Al llegar a Grupo Pacífico me encontré con una empresa familiar que había crecido de forma muy acelerada, pero en cierta medida desorganizada y carente en criterios técnicos. Ya fuere por desconocimiento o falta de tiempo/personal.

En los últimos años anteriores a mí, se habían realizado muchas mejoras, pero estas seguían siendo insuficientes.

La principal tarea encomendada (año 2016) fue la de actualizar y securizar los SO del momento (WServer 2003-2008, Ubuntus…). Es una empresa en la que se precisan de centenares de páginas web por año. Al no ser el ‘core’ empresarial, estas se proporcionan a costes ínfimos para el cliente lo cual terminan siendo webs realizadas de la forma más rápida posible y sin mantenimiento alguno, por ende, esto resultaba en ‘defacings’ continuos y entradas por ‘SQL injection’ entre otros.

No solo eso, sino que me encontré con servidores de correo abiertos al mundo, proxys teóricamente internos los cuales cualquiera des de Internet podía utilizar. Incluso algún servidor albergando y sirviendo públicamente cierto contenido ‘no mencionable’ ajeno a la empresa.

Por si fuera poco, no existía ningún tipo segmentación de red, todos los servidores públicos, internos y usuarios corporativos formaban parte de la misma red ‘interno-público-privada’. Más que eso, aunque con excepciones puntuales, parecía ser que los firewalls internos de los nodos no existían. ‘¿Iptables? ¿Qué es eso?’

Algo que sí hicieron mis antiguos compañeros fue el diseñar una infraestructura redundada con 2 CPDs Activo/Pasivo (ubicados en las propias sedes), utilizando MPLS/Macrolan 100Mbps de Telefónica entre ellos y VPNIP sobre FTTH para las sedes más pequeñas. Los recursos públicos se servían a través de 2 Dibas 10Mbps de Telefonica con 32 y 16 IPs. Cada uno de los CPD contaba con un cluster de 3 nodos ESXi y una cabina NetApp.

La teoría para asegurar disponibilidad de servicio estaba muy bien, teníamos 2 CPDs conectados entre ellos, 1 cluster ESXi y 1 Diba en cada CPD. Además de replicación de cabinas NetApp por Snapmirror/Snapvault. En la práctica… este escenario era inviable. Por suerte nunca tuvimos que utilizar el CPD secundario, digo por suerte porque;

Supongamos que tenemos una parada de tiempo indeterminado al CPD principal:

-	Las Dibas de cada CPD tienen IPs publicas diferentes, sumado a que tenemos 1K-2K entradas DNS y que éstas apuntan directamente a las IPs públicas en vez de a un alias… Deberíamos de modificar todas esas entradas para que atacasen a las nuevas IPs públicas.
o	Era eso, o pedir que Telefonica publicara las IPs públicas de la Diba del CPD principal a la Diba del CPD secundario. Telefonica te da un mínimo de 48 horas para realizar este cambio.
-	Cómo ya he mencionado, todos los servidores están en la misma red que los usuarios. Al tener una conexión por L3 se hubiera tenido que modificar todas las IPs internas de los servidores para ser compatibles con el segmento de red del CPD secundario. Hay que tener en cuenta que no es sólo modificar la IP de los nodos, cada servicio (IIS, Apache…) depende de otros servicios (SQL, MariaDB, SMTP, etc, etc, etc). Lo cual comportaría modificar el 100% de servicios para configurarlos con el nuevo rango de red.
-	Recrear entradas NAT…

-	¿Sabemos cuánto durara esta parada?

-	Y posteriormente, ¿cómo hacemos un rollback?

Bien, aprovechando el aprovisionamiento de servidores nuevos (WServer 2012R2, 2016, Centos 7) mencionado al inicio, éstos ya se implementaron con un mínimo de ‘hardering’ y segmentados en 2 nuevas redes, DMZ e Internal.

Estos no tenían mucho de especial; seguir las guías básicas de ‘PCI-DDS Compliance’ del momento (ej: https://static.open-scap.org/ssg-guides/ssg-centos7-guide-pci-dss.html), configurar firewall interno con src y dst, un OSSEC + mod_security + chroot para servidores web y para de contar. Al final se trataba de bloquear el máximo de ataques posibles y asegurar tanto por firewall interno como perimetral el acceso no indebido a servicios.

Ya tenemos segmentación de red y una securización básica. Ahora, ¿Cómo aseguramos disponibilidad del servicio?

Cómo he comentado, disponíamos del hardware necesario, pero no de la infraestructura de comunicaciones adecuada. Sin contar que el hardware estaba ubicado en 2 edificios de viviendas remodeladas a oficinas.

Rápidamente salió la opción de hacer un ‘colocation’. Después de valorar varios proveedores, cumplimientos técnicos, precios… nos decantamos por BitNap Barcelona y Cogent Madrid.

Cogent Networks nos proporcionó 2 líneas a 1Gbps en cada CPD con un ASN/24 y Full BGP4. Así podíamos tener redundancia de líneas y publicar todo el ASN en cualquiera de los CPDs simultáneamente.

El siguiente paso necesario era el de conectar por L2 no QinQ ambos CPDs con una línea de 10Gbps. En ese instante disponíamos de 2 CPDs con redundancia pública y que, al estar conectados por L2, disponían/veían los mismos segmentos de red. Es decir, podía arrancar un mismo nodo en cualquiera de los CPDs sin tener que hacer modificación alguna.

Para interconectar las sedes y los CPDs se decidió utilizar DMPV sobre tecnologías y proveedores distintos (Radio, FTTH, 4G…). Al final obtuvimos unas latencias de 20-22ms entre sedes y CPDs. Cabe decir que Telefonica no tiene ‘peering’ con Cogent, lo cual, si se utiliza, las latencias suben a ~60ms. Personalmente, Telefonica queda prácticamente descartado en cualquier futuro proyecto. Son comerciales, no técnicos.

El esquema sería algo así:
 

O siendo más específicos:
 

A partir de ahí, conectamos todo el entorno ESXi principal a 10G usando 10GBase-SR (LC).
-	Firewall perimetral
-	Switching
-	ESXi
-	Cabina

Aun así tanto la cabina cómo los ESXi del CPD secundario no disponían de interfaces 10G, al tratarse un CPD de contingencia o para desarrollo no vimos necesario invertir más en ello.

Al final, hablando de costes, supuso un extra de un 15% en la facturación mensual, pero… ¿Qué le cuesta a una empresa que organiza eventos tener todos sus servicios parados durante más de 48 horas? ¿Y si durante esas 48horas se está celebrando un evento de 10.000 personas?

Volviendo al apartado más SysAdmin, con el entorno físico y de networking asegurado, podemos empezar a hablar de automatización de procesos, DevOps, logging…

Una de las herramientas más usadas para asegurar una buena configuración de forma periódica, es el uso de Ansible. No lo utilizamos para despliegues, ya que para ello no hay nada mejor que Docker. Con Ansible nos cercioramos de forma fácil y eficaz de que los servicios más esenciales estén correctamente configurados y arrancados en cualquiera de los nodos, séase; firewalld, ssh, selinux… Al mismo tiempo nos permite administrar de forma muy eficaz los usuarios de los sistemas y servicios como MySQL, etc. O por ejemplo ejecutar actualizaciones de paquetería de forma masiva pero controlada.

La guinda del pastel.

A mi parecer, quien no sepa qué es Docker y sus vertientes (Kubernetes, etc), … debería de aprenderlo. It’s a MUST. No he visto nunca forma mejor y a coste 0 de:
-	Despliegues en masa.
-	Autorecuperación, actualización, escalado y aislamiento de servicios.
-	Creación de entornos HA.
-	Automatización de deploys para los programadores.
-	Pruebas tipo A/B, canary release…
-	Te aseguras que; si te funciona en el entorno de desarrollo, te va a funcionar en el productivo.
-	Mil y una cosas más.

Si una empresa de desarrollo de software no utiliza tecnologías como Docker, está malgastando su tiempo. No hay palabras de elogio suficientes para describir qué aporta Dcoker hasta que lo ves. Y bueno, si ya utilizas Docker, qué te voy a contar…

En el caso que nos acomete, Docker nos proporcionó una fluidez inimaginable a la hora de ‘deployear’ servicios y software propio. Ya no existe el término:

> “A las 23:00h se va a realizar una tarea de mantenimiento bla bla bla”.

Arrancamos una actualización de servicios en segundos, tantas instancias del mismo como queramos y en cualquier nodo disponible. Todo esto sin disrupción alguna del servicio antiguo hasta que el nuevo no está funcionando.

Antes, tratábamos a los servidores como mascotas, los mimabas a más no poder. Actualmente los servidores no son más que un rebaño automatizado.

Por no hablar de monitorización y logging de procesos. Cualquiera de las herramientas tipo ELK, TIG, u otras, te monitoriza servicio a servicio (contenedor a contenedor), por no hablar de proyectos como Netdata que lo lanzas en todos los nodos de tu cluster en lo que dura un parpadeo.

¿Qué tardas desde que un programador hace un commit inicial para un nuevo proyecto hasta que es accesible al mundo?
-	Creas y configuras un servidor de desarrollo y otro para producción.
-	Build de los assets o lo necesario. Artifacts.
-	Publicas en el servidor.
-	Reinicias servicios.

Sí, tienes herramientas que te lo hacen todo. Terraform, Vagrant, Packer, Ansible, Jenkins, Azure DevOps, TeamCity, y un sin fin más.

¿Una empresa con centenares de páginas por año se puede permitir a bajo coste tener centenares de Centos7 (o lo que quieras)? O también, puedes tener unos pocos servidores con decenas de páginas web en ellos, ¿Verdad? No gracias.

En mi caso, soy un enamorado de Gitlab-CI. Te lo hace absolutamente todo.
-	Build de los assets o lo necesario.
-	Creación de imagen de Docker.
-	Publicación del servicio sin disrupción a tu cluster de Docker.

Sólo necesitas un SO, Docker y Gitlab. Eso si, por favor, guardar las sesiones en Redis o parecidos…

