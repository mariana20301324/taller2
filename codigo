from machine import Pin, ADC, PWM # señal de control que modula el ancho del pulso para regular la potencia o velocidad de un dispositivo
import time

# --- 1. Configuración de Pines ---
# Sensores Analógicos (ADC)
lm35 = ADC(Pin(34))#sensopr de temperatuta 25°C 45°C
pot = ADC(Pin(35))#potenciometro regula
ldr = ADC(Pin(32))#fotoresistencia

# Configurar atenuación para leer hasta 3.3V (rango completo del ESP32)
lm35.atten(ADC.ATTN_11DB)#ADC convertidor analogo digital,osea lee los sensores DE 0-3.3 V
pot.atten(ADC.ATTN_11DB)
ldr.atten(ADC.ATTN_11DB)

# LEDs
led_rojo = Pin(21, Pin.OUT)
led_amarillo = Pin(18, Pin.OUT)
led_verde = Pin(23, Pin.OUT)

# Servomotor (Configurado a 50Hz)es un motor electrico diseñado para movimientos precisos y controlados
servo = PWM(Pin(25), freq=50)

# Pulsador (Interrupción con resistencia Pull-Up interna) #interrumptor que desvia la corriente electrica
boton = Pin(19, Pin.IN, Pin.PULL_UP)

# --- 2. Variables Globales ---
sistema_habilitado = False
ultimo_tiempo = 0

# --- 3. Funciones del Sistema ---

# Función de interrupción para habilitar/deshabilitar el sistema
def toggle_sistema(pin):
    global sistema_habilitado, ultimo_tiempo
    tiempo_actual = time.ticks_ms()
    # Antirrebote (debounce) de 300 ms para evitar múltiples lecturas
    if time.ticks_diff(tiempo_actual, ultimo_tiempo) > 300:
        sistema_habilitado = not sistema_habilitado
        ultimo_tiempo = tiempo_actual

# Configurar interrupción en el botón
boton.irq(trigger=Pin.IRQ_FALLING, handler=toggle_sistema)

# Función para mover el servomotor
def mover_servo(angulo):
    # En ESP32 y Wokwi: Duty 40 = 0 grados, Duty 115 = 180 grados aprox.
    if angulo == 0:
        servo.duty(40)
    elif angulo == 180:
        servo.duty(115)

# --- 4. Bucle Principal ---
while True:
    if sistema_habilitado:
        # A. Medición de Temperatura (Sensor LM35)
        # 10mV por cada grado Celsius
        val_lm35 = lm35.read()
        voltaje_lm35 = (val_lm35 / 4095.0) * 3.3
        temperatura = voltaje_lm35 * 100.0 

        # B. Ajuste de Temperatura de Referencia (Potenciómetro)
        # Mapeo de valores del ADC (0-4095) al rango de 25°C a 45°C
        val_pot = pot.read()
        referencia = 25.0 + (val_pot / 4095.0) * 20.0 

        # C. Control del Servomotor
        # Si Temperatura < Referencia -> 0 grados, sino -> 180 grados
        if temperatura < referencia:
            mover_servo(0) 
        else:
            mover_servo(180) 

        # D. Medición de Luz y LEDs (Sensor LDR)
        val_ldr = ldr.read()
        
        # Apagar todos los LEDs antes de la validación
        led_rojo.value(0)
        led_amarillo.value(0)
        led_verde.value(0)

        # Clasificación del nivel de luz (Basado en el rango 0 - 4095)
        if val_ldr < 1365:
            led_rojo.value(1)       # Oscuro 
        elif val_ldr < 2730:
            led_amarillo.value(1)   # Luz media 
        else:
            led_verde.value(1)      # Alta iluminación 

        # Imprimir datos en consola para depuración
        print("Temp: {:.1f}C | Ref: {:.1f}C | Luz: {}".format(temperatura, referencia, val_ldr))

    else:
        # E. Sistema Deshabilitado
        # Servomotor a 0 grados y apagar todos los LEDs
        mover_servo(0) 
        led_rojo.value(0)
        led_amarillo.value(0)
        led_verde.value(0)

    # Pequeña pausa para estabilizar la simulación
    time.sleep(0.2) #time es cuanto tiempo se demora en encender
