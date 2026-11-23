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

<img width="2831" height="1956" alt="maquina_expendedora" src="https://github.com/user-attachments/assets/1d907388-2e86-4739-acd9-63763ba02899" />


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

El estado de Admin se basa en un estado compuesto por otra FSM con un menu de acciones, para poder navegar dentro del menu utilizamos el joystick (eje arriba / abajo) y cuando estamos dentro de una opcion para poder volver al menu principal con el joystick apuntamos a la izquierda:
  - Ver Temperatura: Muestra la temperatura y humedad captado por el sensor DHT
  - Ver distancia sensor: Muestra la distancia captada por el sensor de ultrasonido
  - Ver contador: Muestra el tiempo segundo a segundo del tiempo que ha pasado desde que inicio el programa
  - Modificar precio: Muestra el menu de los diferente productos disponibles, una vez que con el switch del joystick seleccionamos un producto apuntando su eje hacia arriba o hacia abajo podemos sumar o restar el precio del producto en 0.5$ y cuando con el switch confirmamos el cambio permanece los cambios.

Para salir del estado admin tenemos que pulsar el boton externo durante más de 5 sg y transiciona al estado Service_PRESENCE

### Control de antirrebotes del boton externo

Con el botón externo podemos forzar cambios de estado como Service_PRODUCTS -> Service_PRESENCE o la entrada y salida del estado Admin.

Para poder detectar al botón externo tenemos definida una interrupción hardware en modo CHANGE que lo que hace es que se activa cuando hay un cambio en la señal leida por el botón.

Dentro de esta interrupción guardamos los instantes en los que se ha pulsado el botón y en el que se suelta. Una vez guardados los instantes desactivamos la interrupción y ponemos a false un booleano que nos ayudará a saber si la señal está procesada. La desactivamos ya que con los rebotes lo que hace es activar interrupciones, y al comportarse las interrupciones de forma secuencial lo que hace es añadir latencias ya que se van a ir ejecutando una detrás de otra.

Una vez que ya está desactivada tenemos un thread que se ejecuta cada 100 ms ejecutando la función callback_thread1. Dentro de esa función podriamos dividir su funcionamiento en dos partes. La primea parte que es de procesamiento de los tiempos en los cuales se lleva a cabo las transiciones entre estados, aquí es donde utilizaremos la variable booleana ya que es posible que por ejemplo 
(released_button - pressed_button) > 5000 esta condición se ve para entrar y salir de Admin, por lo que si se cumple y entra a Admin al no actualizar los valores de ambas variables automáticamente saldría de Admin, pero como la variable unicamente se actualiza de nuevo a false en la interrupción significa que uno de los dos valores han sido actualizados, por lo que ya no se cumpliría la condición o si se cumple significa que de verdad tiene que salir.
Las condiciones son:
  - Para entrar en Admin:
```
  (released_button - pressed_button) > 5000 && state == Admin && !procesed_signal)
```

  - Para poder reiniciar Service:
```
(state == Service_PRODUCTS) && (released_button - pressed_button) > 2000 && (released_button - pressed_button) < 3000 && !procesed_signal)
```

  - Para salir de Admin
```
(released_button - pressed_button) > 5000 && !procesed_signal)
```

La segunda parte es la del procesamiento de la señal por la cual si la señal se mantiene constante durante dos ejecuciones del thread podemos considerar que no hay rebotes por lo que podemos volver a habilitar de nuevo la interrupción

## Video Demostrativo



https://github.com/user-attachments/assets/8d2d1900-ac6c-4d97-9d7e-49bc3a3d7239


