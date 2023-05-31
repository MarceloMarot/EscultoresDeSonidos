# Entendiendo el Boss OC-2 Octave

## Introducción

El pedal octavador Boss OC-2 es un efecto analógico para guitarra eléctrica que genera dos sub-octavas sobre una señal monofónica de entrada. Este efecto salió a la venta en 1982 y se ha vuelto bastante popular también entre bajistas, violinistas, etc.

Para entender su funcionamiento se analizaron los aspectos más importantes de su esquema circuital original y se
simuló con el programa LTSpice. Se dejaron de lado aquellas secciones del circuito que no son exclusivas de este efecto: buffers, circuito de enclavamiento, conmutación analógica, etc. También se reemplazaron los componentes no disponibles por otros parecidos.

## Amplificacion de entrada

<image src="Imagenes/3-amplificacion entrada.png" alt="3-amplificacion entrada.png" >

La primera etapa es un amplificador operacional en configuración no inversora que proporciona una ganancia de 4,7 para todas las frecuencias de audio y que está acoplado en alterna por entrada y por salida. De esta etapa se obtiene la señal limpia (*clean*) que se mezcla a la salida permitiendo su boosteo y que usan tambien los otros bloques.
Debido a esta amplificación hay que prestar atención de no preamplificar mucho la señal de entrada para que el operacional no recorte por saturación. Por ejemplo, si asumimos que la excursión máxima que permite el operacional ronda los 3V de pico entonces la entrada del pedal debe mantenerse por debajo de los 600mV pico para evitar deformación extra. El amplificador operacional usado para todas las etapas es el TL022, un amplificador doble.

## Detección de frecuencia fundamental y Octavado lógico

El octavador OC-2 se basa en la detección de la frecuencia de entrada y la creación de una onda cuadrada que permita implementar la división de frecuencia mediante flip-flops. Esta operación se puede dividir en tres partes, como se explica a continuación.

<image src="Imagenes/4-deteccion fundamental.png" alt="4-deteccion fundamental.png" >

### Filtrado de entrada

El primer paso de este bloque es minimizar las frecuencias más altas. Para esto la señal pasa por un filtro RC pasa bajos de 1º orden y frecuencia 720Hz,  seguido de un filtro Sallen-Key de segundo orden orden cuya frecuencia de polo es de 700Hz y un factor Q de 1,58. (Un factor Q mayor a 1 provoca un realce alrededor de la frecuencia de polo). El resultado de combinar ambos es un filtro de tercer orden cuya banda de paso alcanza aproximadamente los 700Hz. Este filtro le quita sensibilidad al octavador a las frecuencias de entrada más altas, es decir empuja a “ignorar” las notas más altas.
Luego del filtro hay una red pasiva que añade filtrado paso bajo extra alrededor de 30kHz y además bifurca la señal en dos partes con amplitudes distintas dependientes de la frecuencia. Este es el comportamiento del filtro completo respecto a la frecuencia:

<image src="Imagenes/5-filtro entrada.png" alt="5-filtro entrada.png" >

Se observa como la frecuencia de corte del filtro Sallen Key (la frecuencia del 70% de la entrada) se acomoda alrededor de los 800Hz. Las salidas de la red RC alcanzan la frecuencia de corte un poco antes. También se observa cómo el filtro atenúa dramáticamente las componentes de mayor frecuencia.



### Detección de frecuencia fundamental

El objetivo de este bloque es crear una señal cuadrada de la misma frecuencia que el tono de entrada.
Como primer paso se utilizan dos detectores de pico modificados para ponderar la media del semiciclo positivo y del semiciclo negativo de la señal filtrada (la más atenuada).  Cada detector consiste en un amplificador operacional, dos diodos, dos resistencias y un capacitor. Los diodos y resistencias se conectan de modo de mantener el operacional en su zona activa en todo momento, previniendo la saturación. La carga del capacitor se realiza a través de una única resistencia de 1k en tanto que la descarga se realiza a través de la serie de las dos resistencias, con un valor equivalente once veces mayor. El resultado es una tensión más o menos continua que es proporcional al pico de tensión de entrada pero algo inferior en amplitud. Los transitorios de carga y de descarga tienden a ser exponenciales decrecientes. La constante de tiempo de carga es de 1 milisegundo en tanto que la constante de descarga es de 11 milisegundos.

<image src="Imagenes/6-medias ponderadas.png" alt="6-medias ponderadas.png" >

En segundo lugar la señal filtrada de entrada (la componente mayor) se compara con las dos medias ponderadas utilizando amplificadores operacionales. El haber usado dos componentes de señal de distintas amplitudes permite garantizar la conmutación en cada ciclo y da algo de margen ante los ruidos e interferencias.
Cada operacional crea un pulso destinado a excitar los pines de set y reset de un flip-flop: el pulso de set fuerza la salida Q al nivel alto (“1”)  y el pulso en reset fuerza la salida Q al nivel bajo (“0”). La salida Q¯ (Q negada) es exactamente la inversa.
En la gráfica siguiente se muestra la respuesta de estos detectores y los pulsos creados ante una sinusoide de 200Hz:

<image src="Imagenes/7-pulsos.png" alt="7-pulsos.png" >

El separar la señal procedente del filtro ayuda a asegurar la conmutación en el caso de haber ruidos de fondo y también a darle más anchura a los pulsos de salida.
La salida del flip-flop resultante es una onda cuadrada con la misma frecuencia de la señal de entrada. El flip-flop usado en el esquema original es la unidad B de un CD4013, un integrado con dos flip-flop tipo D.  A esta unidad se le conectan los pines de datos (D) y de reloj (clock, CK) a 0 para que no interfieran con la creación de la señal de salida. Este es el diagrama de tiempos:

<image src="Imagenes/8-fundamental.png" alt="8-fundamental.png" >

Obsérvese como los flancos de subida de set (naranja) y de reset (cian) marcan el cambio de estado de la señal fundamental (roja).
Este sistema de deteccion sólo funciona bien ante señales monofónicas, es decir no es capaz de detectar correctamente la componente fundamental de los acordes . Ante la presencia de señales polifónicas el resultado es una onda rectangular con periodo inestable y el octavado da lugar a señales que al oído se perciben como disruptivas y que pueden ser molestas.

### Octavado lógico

Los flip-flops permiten crear señales cuadradas de la mitad de la frecuencia de entrada. Para hacer la primera división de frecuencia se usa la segunda unidad del CD4013. En esta etapa las entradas set y reset se ponen a cero. En estas condiciones las salidas se actualizan con cada flanco ascendente de la señal de reloj, cambiando de estado de 0 a 1 y viceversa.
Además, como la entrada D se conecta a la salida complementaria Q¯ el resultado es que tanto Q como Q¯ cambian de estado con cada flanco ascendente sobre el clock CK. Y como CK va conectada a la señal fundamental de la etapa previa, entonces este flip-flop cambiará de estado con cada nuevo ciclo de la onda previa, obteniéndose una onda con el doble de período y, por tanto, de la mitad de frecuencia. Esta será la primera octava lógica.
En la presente simulación se lo sustituyó con un flip-flop tipo D.
Así se ve el diagrama de tiempos de los flip-flops:

<image src="Imagenes/9-flip-flops.png" alt="9-flip-flops.png" >

Aquí se nota cómo están sincronizadas en el tiempo las octavas creadas. Estas señales serán usadas más adelante para modelar las formas de onda analógicas que irán a la salida del sistema.
Para conseguir la segunda octava el diseño original usaba un flip-flop T buffereado , el BA634. Éste también hace la división de frecuencia por dos, y al ser excitado con la primera octava el resultado es una segunda octava lógica, es decir una onda cuadrada con la cuarta parte de la frecuencia de entrada.Este chip está obsoleto y en muchos diseños derivados se lo reemplaza con un flip-flop JK como el CD4027 o con un segundo flip-flop D como el CD4013.

## Modulación y conformación de las formas de onda

Una vez detectada la señal fundamental se implementa la conformación de onda de las dos octavas añadidas por el pedal. Este es el subcircuito que las crea:



<image src="Imagenes/11A-creacion primera octava.png" alt="11B-creacion primera octava.png" >

<image src="Imagenes/11B-creacion segunda octava.png" alt="11B-creacion segunda octava.png" >


Este circuito es conformado por dos bloques analógicos casi idénticos en cascada, consistentes cada uno en un desplazador de nivel alimentado por la señal de audio preamplificada seguido de una etapa de conmutación y que pasa finalmente por un filtro paso bajo. Se repasan las etapas paso a paso:

### Desplazamiento de Nivel

El desplazamiento de nivel es conformado por un capacitor de acople y un diodo de germanio conectado a la tensión de referencia. El capacitor se carga a una tensión casi idéntica a la del pico de señal de entrada e idealmente la señal de salida queda corrida de nivel de modo que su valor instantáneo quede comprendido entre cero y el doble del valor pico. Como el cátodo del diodo está conectado a la referencia la tensión se desplaza al positivo. La idea de usar un diodo de germanio en vez de silicio es minimizar la caída de tensión del diodo durante la conducción (*polarización directa*) y que por tanto el desplazamiento de tensión sea más próximo al pico de señal ante entradas débiles. (La etapa de ganancia previa también ayuda a este propósito).
Debido al  breve periodo de conducción del diodo, esta etapa introduce una pequeña distorsión.

### Conmutación

La conmutación se implementa con un amplificador operacional cuya realimentación es afectada por un transistor JFET, cuya puerta (gate) es controlada por los flip-flops mediante un divisor de tensión resistivo con resistencias:

- Cuando el gate es empujado al negativo (0V) el JFET se comporta como un circuito abierto y el operacional se porta como un amplificador no inversor con ganancia global 0,5 aproximadamente;
- Cuando el gate es movido hacia la tensión de referencia por el flip-flop el JFET entra en su zona óhmica en la cual el componente “se cierra”  referenciando la entrada no inversora a masa. Gracias a ello el operacional se comporta como un amplificador inversor y la ganancia es -0,5 aproximadamente.

El resultado es una señal que intenta parecerse a la original en amplitud y forma pero cuya frecuencia está justo una octava abajo. Para mejorar lo más posible la forma de onda de salida la conmutación debe producirse lo más cercana posible al cero de la entrada, de este modo se minimiza la formación de los armónicos más agudos y agresivos al oído.

### Filtrado

Cada señal de octava pasa por un filtro paso bajo de tercer orden, conformado por un filtro RC de 1º orden seguido de un Sallen-Key de 2º orden. El objetivo es reducir los armónicos superiores de la octava, sobre todo aquellos producidos por la conmutación. Los valores no son iguales para ambos filtros:

- para la primera octava el filtro RC es de 330Hz y el filtro SK tiene frecuencia de polo 480Hz y factor Q 2,3;
- Para la segunda octava el filtro RC es de 150Hz y el filtro SK tiene frecuencia de polo 150Hz y factor Q 1,6.

Se muestra la respuesta de esos filtros respecto a la frecuencia:

<image src="Imagenes/12-filtros salida.png" alt="12-filtros salida.png" >

Las frecuencias diseño están elegidas para que actúen atenuando las componentes armónicas de las ondas creadas de manera proporcional.

## Formas de Onda

En la gráfica se muestra la respuesta de los tres circuitos de la primera octava ante un tono puro de entrada de 200Hz. Se acopla la señal de entrada preamplificada (cian) al desplazador de nivel (magenta) y se conmuta en base a la primera octava lógica (amarillo), que se muestra al doble de amplitud:

<image src="Imagenes/13A-octava 1.png" alt="13A-octava 1.png" >

El resultado final se parece mucho al de una sinusoide que pasa por una distorsión de cruce (crossover distortion) en el centro lo cual se traduce en distorsión armónica agregada.
Al filtrar (rojo) se suaviza sutilmente la forma de onda:

<image src="Imagenes/13B-octava 1.png" alt="13B-octava 1.png" >

Como la conmutación se produjo muy cerca del cero las discontinuidades producidas son pequeñas, esto se traduce en armónicos de alta frecuencia con poca amplitud y entonces el filtro de salida apenas afecta la forma de onda. Todo esto se verifica con la representación en respecto a la frecuencia:

<image src="Imagenes/14-octava 1-fft_lin.png" alt="14-octava 1-fft_lin.png" >

En esta gráfica se nota cómo la fundamental de salida está justo una octava por debajo de la entrada (100Hz), y además como la distorsión se concentra en el tercer armónico (en este caso a 300Hz) y en mucha menor medida en el quinto armónico (500Hz).
El proceso se repite respecto a la segunda octava. Se toma la primera octava (rojo), se pasa por el desplazamiento de nivel (cian) y se hace la conmutación (naranja) 
<image src="Imagenes/15A-octava 2.png" alt="15A-octava 2.png" >

En este caso se nota una mayor deformidad en la forma de onda de salida de la segunda octava respecto al caso de la primera y con discontinuidades muy grandes debidas a la conmutación, que está muy alejada del valle de la onda desplazada. El filtro esta vez hace un suavizado mucho más agresivo (gris claro):

<image src="Imagenes/15B-octava 2.png" alt="15B-octava 2.png" >

La representacion en el dominio de la frecuencia es el siguiente:

<image src="Imagenes/16-octava 2-fft_lin.png" alt="16-octava 2-fft_lin.png" >

Esta vez la presencia de armónicos superiores al quinto es mucho más evidente. Con la ayuda del filtro el contenido de armónicos sobre esta señal se reduce dramáticamente a partir del quinto armónico (250Hz), respetando únicamente el tercer armónico (150Hz) .
Un detalle interesante de este efecto es que todas las señales que entrega  a su salida tienen una amplitud pico de salida parecida:

<image src="Imagenes/17-Todas las salidas.png" alt="17-Todas las salidas.png" >

Y las componentes de frecuencia de cada señal de salida serían como la siguiente:

<image src="Imagenes/18-todas las salidas-fft.png" alt="18-todas las salidas-fft.png" >

## Mezcla de salida:

<image src="Imagenes/19-mezcla salida.png" alt="19-mezcla salida.png" >

Cada señal de salida (*clean, 1º octava y 2º octava*) tiene su propio potenciometro de volumen con perfil logarítmico y valor maximo 100k. Estos potenciómetros proporcionan en la posicion intermedia del cursor deslizante una amplitud de alrededor del 20% de la amplitud de entrada. La mezcla dse realiza uniendo los contactos deslizantes de los potenciómetros mediante resistencias de 100k,  dando lugar en el punto medio a un promedio de las tres componentes. Dado que son tres componentes mezclándose, cada señal puede alcanzar a la salida una amplitud máxima de un tercio de la señal de entrada. Y si se recuerda que la señal de entrada es amplificada 4,7 veces al entrar al efecto, la amplitud máxima de cada componente por separado es (4,7/3) = 1,6 veces la amplitud de entrada, permitiendo el uso del efecto como booster.
Es interesante estimar la amplitud de cada componente de salida del mezclador en base a la posicion de los potenciómetros.

- Colocándolo al mínimo la componente se anula prácticamente por completo;
- Si se coloca un potenciómetro a medio recorrido (*a las 12*) su componente tendrá un tercio de la amplitud de entrada (el 20% del máximo);
- para igualar la amplitud de entrada el cursor debe estar aproximadamente *entre las 2 y las 3*;
- Colocándolo al máximo la componente sube a 1,6 veces. La potencia es más del doble de la potencia de entrada (2,5 veces).

La amplitud final de salida será la suma de las tres componentes.

## Resultados y Conclusiones

Como resumen, se puede afirmar que el Boss OC2 intenta imitar la forma de entrada al octavar; sin embargo su filtrado agresivo da lugar a una pobre variedad de armónicos de salida y ello redunda en un sonido de octavas que se percibe como sintético y opaco. Sin embargo esto se compensa con el añadido de la señal original, dando lugar a un sonido similar al original pero con mayor presencia y profundidad.

## Comentarios y especulaciones varias

- Para reemplazar los JFET's es deseable usar transistores de bajo VGS(off) para asegurar una buena conmutación. También se podría usar como reemplazo conmutadores CMOS integrados como el CD4016 (canal simple, cuatro etapas) o el CD4053 (conmutador, tres etapas) controlándolos con los flip-flops directamente.
- Los TL022 pueden ser sustituidos sin problemas por los TL072 y TL082, los cuales tienen igual pinout y mejores características: son más veloces en conmutación, tienen mucho más ancho de banda y un poco mejor rango de salida. Si se busca hacer un rediseño de PCB podrían usarse los TL074 ó TL084, los cuales son la versión cuádruple de los anteriores. De usar otras opciones, es importante que los operacionales que exciten al flip-flop tengan un rango de salida que pueda alcanzar entre 2V y 7V como minimo para asegurar que los flip-flops respondan bien.
- Cabe señalar tambien que los opamps (y más aun, los TL022) no son los componentes ideales para comparar debido a que la ganancia interna decae dramáticamente con la frecuencia y además estos componentes tienen cierto retardo para salir de la saturación. Posiblemente por ello se haya colocado el filtrado previo a la detección a 700Hz de modo de minimizar los problemas de falta de ganancia y de conmutacion a costa de impedir el seguimiento  de frecuencia ante las notas más agudas. Como dato de referencia: en el traste 13 de la primera cuerda de la guitarra se alcanzan los 698Hz, usando afinación estándar. Los problemas de seguimiento pueden hacerse  evidentes en trastes superiores. En caso de usar opamps más modernos como los TL07X o TL08X se dispone de más ganancia en altas frecuencias por lo que podría aumentarse la frecuencia de corte y así *estirar* la frecuencia máxima de seguimiento hasta 1,3kHz (traste 24) sin problemas añadidos.
- Los diodos de germanio podrían ser reemplazados por diodos Schottky para lograr un desempeño parecido. 
- Este pedal suele tener problemas para trackear las señales más graves. Esto podría paliarse aumentando el valor de las resistencias R33 y R38 para aumentar el tiempo de descarga de los detectores de media. Otra opción es aumentar los valores de C22 y C23, sin embargo esto incrementa también el tiempo de subida de estos circuitos. Una tercera opción es aumentar  C18 (por ejemplo a 47n), lo cual “separa” mejor las dos señales de entrada al detector de fundamental en las frecuencias inferiores (ver el análisis del primer filtro). Se requiere experimentación.
- La señal de referencia para crear la segunda octava procede del filtro de la primera octava. Esta manera permite minimizar los problemas de replicado de frecuencias (*aliasing*) al hacer la conmutación de la segunda octava. Sin embargo , para crear la primera octava la señal de referencia se toma de la entrada sin filtrar. No está claro el motivo de esta decisión.
- Los filtros introducen un desfase que depende de la frecuencia. En el caso del filtro de 3º orden el desfase puede alcanzar hasta 270º (90º x 3) en las frecuencias más altas y la red RC posterior añade un pequeño desfase adicional. Sin embargo en este diseño parece no dársele demasiada importancia a esta cuestión. A continuación se ven las respuestas en fase de todos los filtros:

<image src="Imagenes/20-fase filtros.png" alt="20-filtro entrada-fase.png" >

- En el Boss OC2 la conmutación de bypass sólo se hace a la salida, por ello el circuito interno trabaja incluso con el bypass activado. Probablemente debido a que la señal de entrada no sufre grandes amplificaciones d
Esto contrasta


