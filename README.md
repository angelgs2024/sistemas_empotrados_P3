# Practica 3

## Objetivo
En esta práctica lo que buscamos es desarrollar un sistema que simule el comportamiento de una máquina expendedora, para poder poner en práctica librerias como ArduinoThread, TimerOne y librerias especificas de componentes.

## Componentes
  - Arduino UNO
  - Pantalla LCD
  - Módulo Joystick
  - Sensor de Temperatura y Humedad DHT11.
  - Sensor de Distancia por Ultrasonidos
  - Pulsador / Botón externo.
  - 2 LEDs estándar: LED verde y LED rojo

## Imagen montaje

<img width="2831" height="1956" alt="maquina_expendedora" src="https://github.com/user-attachments/assets/662a7a61-f19c-4342-a87a-0448d8a03183" />

## Funcionamiento

Para poder llevar a cabo el funcionamiento deseado he desarrollado una FSM basada en 4 estados principales:

### Boot

El estado boot es el estado de arranque en el cual lo que hacemos es encender 3 veces el led verde. Hacemos lucir 3 veces el led verde a traves de Timer1, definimos para que Timer 1 ejecute interrupcion_time cada segundo. Dentro de esta funcion guardamos una variable contador que va aumentando en una unidad la cual nos va a ayudar para poder ir guardando los segundos que han pasado desde que empezó el programa.

También vamos modificando el valor de una variable booleana que es la que hace que se vaya encendiendo y apagando el led. Si es true encendemos y si es false lo apagamos.

Mientras luce el led en la pantalla LCD imprimimos "Cargando ..."

Una vez que el led acaba de lucir transicionamos a Service_PRESENCE

### Service_PRESENCE
En Service_PRESENCE lo que hacemos es esperar a un cliente, para ello lo que hacemos es mientras que la distancia que lee el sensor no sea menor que 100 el LCD imprime "ESPERANDO CLIENTE".

Mientras que no se detecta a nadie también ejecutamos el controller que va comprobando posibles transiciones al estado de Admin si el botón externo ha sido pulsado por más de 5 segundos seguidos. En caso de que se detecte a alguien y no se haya cambiado a Admin lo que hacemos es transicionar a Service_PRODUCTS

### Service_PRODUCTS

En el estado de Service_PRODUCTS lo que vamos a hacer es durante 5 segundos vamos a imprimir los valores que capta el sensor DHT tanto temperatura como humedad. Controlamos el tiempo que ha pasado a través del uso de micros().

Después de enseñar la temperatura lo que hacemos es que vamos a enseñar el menu de productos disponible. Vamos a ir navegando gracias al joystick (eje arriba / abajo) y para confirmar la selección del producto utilizaremos el switch del joystick.

Una vez seleccionado el producto lo que hacemos es imprimir durante un número aleatorio entre 4sg y 8sg "Preparando Cafe..." durante ese tiempo el led rojo va a ir aumentando gradualmente su intensidad mostrando el progreso del proceso. Para saber que intensidad tiene que tener lo que hacemos es calcular la intensidad por cada microsegundo mediante 255 / tiempo aleatorio y luego segun el tiempo que ha pasado lo multiplicamos por la intensidad.

Una vez que ha acabado de imprimir el mensaje durante el numero aleatorio de segundos lo que hacemos es volver a Service_PRESENCE salvo que durante la ejecución del controlador el thread haya modificado el estado a Admin.

Durante Service_PRODUCTS si el botón externo es pulsado entre 2 y 3 segundos el sistema transiciona de nuevo a Service_PRESENCE y si el botón externo ha sido pulsado por más de 5 segundos seguidos el sistema transiciona a Admin.

### Admin
