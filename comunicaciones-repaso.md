# COMPILADO DE REPASO PARA EL PARCIAL

---

### 1. SATELITES Y ORBITAS

* **Pregunta:** ¿Cuál es el problema más crítico que presentan las comunicaciones de datos en satélites geoestacionarios (GEO)?
  * **Respuesta:** Alta latencia o demora en la propagación de la señal.
  * **Explicación:** Los satélites GEO están muy lejos (~35.786 km), por lo que la señal tarda cerca de medio segundo ($500-600\text{ ms}$) en ir y volver.

* **Pregunta:** ¿Qué desventaja fundamental tienen los satélites de órbita baja (LEO) respecto de los geoestacionarios?
  * **Respuesta:** Se necesitan más satélites para cubrir el servicio las 24 hs del día.
  * **Explicación:** Los satélites LEO se mueven rápido y tienen un campo de visión pequeño. Para cubrir toda la Tierra sin interrupción se requieren constelaciones de cientos o miles de satélites.

* **Pregunta:** ¿Qué afirmación es correcta al comparar LEO con GEO?
  * **Respuesta:** D. Ninguna de las opciones. (Las opciones A, B y C del Tema 2 eran falsas: LEO tiene MENOR latencia, MENOR potencia por satélite y requiere MÁS satélites).

* **Pregunta:** Los subsistemas de comunicaciones o payload (carga útil) de un satélite están compuestos por:
  * **Respuesta:** Antenas, transponder y amplificadores de potencia.
  * **Explicación:** El payload es el equipo encargado exclusivamente de recibir, procesar, amplificar y retransmitir las señales de comunicación.

---

### 2. MEDIOS GUIADOS: FIBRA OPTICA

* **Pregunta:** Al comparar fibra óptica monomodo respecto de multimodo:
  * **Respuesta:** La monomodo tiene menor dispersión del pulso de luz que la multimodo.
  * **Explicación:** En monomodo la luz viaja por un solo camino (un único modo), eliminando la dispersión intermodal y permitiendo mayor distancia y velocidad.

* **Pregunta:** ¿Cuál es la tercera ventana de transmisión en fibra óptica y qué característica tiene?
  * **Respuesta:** La tercera ventana tiene menor atenuación que las otras dos (está ubicada en los $1550\text{ nm}$).

* **Pregunta:** En la fibra óptica, ¿en qué longitud de onda ocurre la segunda ventana?
  * **Respuesta:** $1300\text{ nm}$ (específicamente $1310\text{ nm}$).

* **Pregunta:** ¿Qué efecto produce la dispersión del pulso de luz en una fibra óptica?
  * **Respuesta:** El ensanchamiento del pulso a la salida de la fibra óptica (lo que limita el ancho de banda / velocidad máxima).

* **Pregunta:** ¿Qué tipo de cable se utiliza en los cables submarinos actuales?
  * **Respuesta:** Fibra óptica.

* **Pregunta:** ¿La dispersión cromática en la fibra óptica a qué afecta?
  * **Respuesta:** Afecta el ancho de banda de la fibra óptica al ensanchar el pulso de luz a la salida.

---

### 3. MEDIOS GUIADOS: CABLE COAXIL Y UTP

* **Pregunta:** El cable coaxil, ¿es un par balanceado?
  * **Respuesta:** No (es un medio desbalanceado).

* **Pregunta:** La impedancia característica de un cable coaxil depende de:
  * **Respuesta:** De las dimensiones del cable y de la constante dieléctrica.
  * **Explicación:** NO depende de la frecuencia (es constante en altas frecuencias) ni de la longitud.

* **Pregunta:** La impedancia característica en un cable coaxil NO está relacionada con:
  * **Respuesta:** La frecuencia.

* **Pregunta:** ¿A qué tipo de medio de comunicación alámbrico NO le afectan las interferencias electromagnéticas (EMI)?
  * **Respuesta:** D. Ninguna de las opciones.
  * **Explicación:** A los cables metálicos (UTP, Coaxil, Multipar) sí les afectan. El único inmune es la fibra óptica (que no figuraba en la lista).

* **Pregunta:** En el cable UTP, ¿de qué depende principalmente su categoría?
  * **Respuesta:** Del trenzado de los pares (cuanto más cerrado/apretado es el trenzado, mayor es la categoría y menor la diafonía).

* **Pregunta:** ¿Cómo se denomina el acoplamiento no deseado (interferencia) entre un par de cables y otro dentro de un cable UTP?
  * **Respuesta:** NEXT o paradiafonía (Near-End Crosstalk).

* **Pregunta:** ¿Propagation delay y delay skew representan el mismo fenómeno en el cable UTP?
  * **Respuesta:** No. (Propagation delay es el tiempo de viaje; delay skew es la diferencia de tiempo de llegada entre los distintos pares).

---

### 4. PROPAGACION Y ANTENAS

* **Pregunta:** ¿Qué modo de propagación emplean los enlaces de microondas y satelitales?
  * **Respuesta:** Propagación por onda directa (Línea de vista / Line of Sight).

* **Pregunta:** Para frecuencias menores a la banda de HF (p. ej., LF/MF como la Radio AM), ¿cómo se produce la transmisión?
  * **Respuesta:** Onda terrestre (o de superficie).

* **Pregunta:** La transmisión electromagnética en la banda de HF ($3$ a $30\text{ MHz}$) es fundamentalmente:
  * **Respuesta:** Onda ionosférica.

* **Pregunta:** En la banda de transmisión UHF ($300\text{ MHz}$ a $3\text{ GHz}$), la propagación es:
  * **Respuesta:** Por onda directa.

* **Pregunta:** ¿Qué rango de frecuencias se refracta en la ionosfera y rebota hacia la tierra?
  * **Respuesta:** El rango de $3$ a $30\text{ MHz}$ (HF), debido a la alta concentración de electrones originada por la radiación solar.

* **Pregunta:** ¿Cuál es la longitud de onda ($\lambda$) para una frecuencia de $100\text{ MHz}$?
  * **Respuesta:** $3\text{ m}$.
  * **Cálculo:** $\lambda = \frac{c}{f} = \frac{3 \cdot 10^8\text{ m/s}}{100 \cdot 10^6\text{ Hz}} = 3\text{ metros}$.

* **Pregunta:** En un enlace de $30\text{ MHz}$, ¿cuál es la longitud necesaria para una antena de media longitud de onda ($\lambda / 2$)?
  * **Respuesta:** $5\text{ m}$.
  * **Cálculo:** $\lambda = \frac{3 \cdot 10^8}{30 \cdot 10^6} = 10\text{ m} \implies \text{Media onda} = \frac{10\text{ m}}{2} = 5\text{ m}$.

* **Pregunta:** ¿Por qué se utiliza la media longitud de onda ($\lambda / 2$) como base para antenas dipolo?
  * **Respuesta:** Porque los efectos capacitivos e inductivos se cancelan, permitiendo que la antena entre en resonancia e irradie con la máxima eficiencia.

---

### 5. MULTIPLEXACION Y MULTIACCESO (FDM, TDM, CDMA, TDMA)

* **Pregunta:** ¿Método que divide el ancho de banda en porciones de frecuencia más pequeñas transmitiendo en simultáneo?
  * **Respuesta:** FDM (Multiplexación por División de Frecuencia).

* **Pregunta:** ¿Método que asigna un código único a cada usuario transmitiendo en la misma frecuencia y al mismo tiempo?
  * **Respuesta:** CDMA (Code Division Multiple Access).

* **Pregunta:** ¿Qué multiplexor se puede utilizar para transmitir tanto señales analógicas como digitales?
  * **Respuesta:** FDM.

* **Pregunta:** ¿Cómo se denomina al dispositivo cuya capacidad de salida es menor que la suma de las capacidades de entrada?
  * **Respuesta:** Concentrador (o multiplexor estadístico / STDM).

* **Pregunta:** En un multiplexor TDMA, ¿puede una estación utilizar todo el ancho de banda del canal?
  * **Respuesta:** Sí.
  * **Explicación:** En TDMA se utiliza todo el ancho de banda, pero únicamente durante el intervalo de tiempo (timeslot) asignado.

---

### 6. DIGITALIZACION Y MODULACION (PCM, PSK, PDH)

* **Pregunta:** Pasos y orden correcto del método de digitalización de señales analógicas (PCM):
  * **Respuesta:** Muestreo, Cuantificación y Codificación.

* **Pregunta:** Dada una señal con frecuencia máxima de $4.000\text{ Hz}$, ¿cuál debe ser la frecuencia de muestreo mínima en PCM?
  * **Respuesta:** $8.000\text{ muestras/seg}$ (o $8\text{ kHz}$).
  * **Explicación:** Teorema de Nyquist: $f_s \ge 2 \cdot f_{\max} = 2 \cdot 4000 = 8000\text{ Hz}$.

* **Pregunta:** ¿Cuál afirmación sobre PCM es correcta?
  * **Respuesta:** Permite optimizar los sistemas de codificación, de seguridad y control de errores.

* **Pregunta:** En el sistema PDH norma europea, ¿cuántos canales transmite el nivel E1 y a qué velocidad opera?
  * **Respuesta:** 32 canales totales ($30$ de voz $+ 2$ de señalización/sincronismo) y opera a $2,048\text{ Mbps}$.

* **Pregunta:** Módem de $2400\text{ baudios}$ multinivel PSK que se quiere transmitir a $9600\text{ bps}$. ¿Qué esquema usar?
  * **Respuesta:** Cuadribits y 16 PSK.
  * **Cálculo:** $\text{Bits por baudio} = \frac{9600\text{ bps}}{2400\text{ baudios}} = 4\text{ bits/baudio}$ (Cuadribit). Los estados requeridos son $2^4 = 16\text{ PSK}$.

* **Pregunta:** En un módem 8 PSK, ¿cuál es el desfasaje entre estados consecutivos?
  * **Respuesta:** $\pi / 4$ (o $45^\circ$).
  * **Cálculo:** $\frac{360^\circ}{8} = 45^\circ \implies \frac{2\pi}{8} = \frac{\pi}{4}$.

* **Pregunta:** En la modulación 4 PSK (QPSK), ¿cuál es el salto de fase y cuántos bits se transmiten por baudio?
  * **Respuesta:** D. Ninguna de las anteriores.
  * **Explicación:** En 4 PSK hay $90^\circ$ de salto de fase y $2\text{ bits}$ por baudio ($2^2=4$).

* **Pregunta:** En una modulación 64 PSK conformada por señal moduladora, portadora y modulada, ¿cuál/es son digitales?
  * **Respuesta:** La moduladora.
  * **Explicación:** La moduladora es el dato digital binario original ($0$s y $1$s). La portadora es una senoide analógica y la señal modulada sigue siendo una onda electromagnética analógica.

---

### FÓRMULAS RÁPIDAS PARA REPASAR
1. $\lambda = \frac{300}{f\text{ [MHz]}}$ (resultado en metros).
2. Antena dipolo de media onda: $L = \frac{\lambda}{2}$.
3. Muestreo de Nyquist: $f_s = 2 \cdot f_{\max}$.
4. Tasa de transmisión: $\text{bps} = \text{baudios} \cdot \text{bits por baudio}$.
5. Estados de modulación: $M = 2^n$ (donde $n$ es la cantidad de bits por baudio).