# Desc
[video](https://www.youtube.com/watch?v=2tIEtS-d1og)

parpadear leds en puerto 0 con delay de 10ms. Mientras leer datos de puerto 1 y mandarlo a puerto 2. Usar interrupcion timer 1 en modo 1.

# Explicacion

## ISR
ISR timer 1 direccion = 001Bh
lo define como
```
ORG 001BH
```
despues de esto escribe las instrucciones de la rutina ISR para que queden guardadas desde 001Bh

## TMOD
inicializar registro TMOD (ver pag 7 fig.6 manual de HW) para dar modo y cual timer
como usa timer 1 usa parte alta
* 1º bit (gate) = 0 porque usamos instrucciones de SW para empezar y parar el timer
* 2º bit (C/T) = 0 porque usamos como timer 
* 3º/4º bit (M1, M0) = 01 para seleccionar modo 1
* los otros 4 bits son para timer 0, asi que los pone en 0
entonces queda en TMOD:
001010000 -> 10h
por lo que que guarda 10h en TMOD para fijar el timer 1 en modo 1, con entrada como timer (no contador)

## TCON
para arrancar y parar el timer usa registro TCON (ver pag 7 fig.8 manual de HW)
* 1º bit, D7 / TCON.5 (TF1) = 0 marca el overflow pero usa interrupciones asi que no importa. Si es 1, es que despues de operar el timer el micro automaticamente va a la direccion 001Bh (donde esta la subrutina de ISR del timer 1) y la corre.
* 2º bit, D6 / TCON.6 (TR1) = 1 para empezar timer 1, 0 para frenar
entonces para arrancar el timer
```
SETB TCON.6
```
entonces queda en TCON:

01000000 -> 40h

## IE
inicializar registro Interrupt Enable.
Como usamos timer 1, hay que inicializar el bit de timer 1
* 1º bit (EA) = 1 para poder usar interrupciones
* bit D3 (ET1) = 1 porque corresponde al timer 1.
* el resto los deja en 0

entonces queda en IE:

10001000 -> 88h

## Contadores
inicializar registro de contador del timer
como usa delay de 10 ms:
* parte alta (TH1) = 0DBh 
* parte baja (TL1) = FFh

En modo 1 el overflow de TH1 setea TF1 y corre la interrupcion, carga TL1 al maximo para que solo pase lo que falta de TH1 para FFh, e inicializa TH1 en DBh??

no entendi que paso aca

# Programa
```
ORG 0000H   ; las instrucciones van a estar en esa direccion
; cuando se resetee el micro va a empezar a correr el programa desde esta direccion, asi que el main deberia volver a esta direccion

LJMP main   ; en vez de escribir el programa desde 0000H usa la etiqueta main en vez de escribir desde ahi para que si es muy largo no sobrepase 001BH que es donde va a estar la ISR del timer

ORG 001BH      ; la direccion donde va a estar la subrutina de interrupcion
CPL A          ; complemente lo que este en el acumulador, cambia  de FFH (led prendido) a 00H (led apagado)
MOV P0, A      ; pasa el dato en A a los puertos con los led (prende/apaga)

CLR TCON.6     ; para el timer
MOV TH1, #0DBH ; inicializo registro contador del timer de nuevo
MOV TL1, #0FFH ; inicializo registro contador del timer de nuevo
RETI           ; termina interrupcion, hace CLR TF1 y vuelve al programa normal

main:
    ; para el timer
    MOV TMOD, #10H ; para timer 1 en modo 1
    MOV TH1, #DBH  ; registro contador del timer parte alta para delay
    MOV TL1, #FFH  ; registro contador del timer parte baja para delay
    MOV IE, #88H   ; interrupt enable
    SETB TCON.6    ; empieza el timer

    ; para prender los led
    MOV A, #0FFH   ; para mandar los datos a los puertos para prender led
    MOV P0, A      ; prende los led
    
    ; para mandar mientras datos de P1 a P2    
back:
    MOV A, P1      ; carga al acum datos de P1
    MOV P2, A      ; manda a P2 los datos de P1
    SJMP back      ; loop de leer de P1 y mandar a P2
```