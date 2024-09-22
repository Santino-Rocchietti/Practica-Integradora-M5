# TITULO
## CONTEXTO DEL PROYECTO 
## Call Center
![N|Solid](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSyWEMAsO2fStc8YIGr9f-co5h7D84aCB_E0A&usqp=CAU)

Analizar las operaciones de un Call Center de un Banco, para proponer mejoras en:
* Eficiencia operativa, proponiendo mejoras operativas.
* Mejorar la satisfacción del cliente, cumpliendo los SLA comprometidos.
* Brindar una herramienta para la gestión y la toma de decisiones a los managers del Call Center.

En conclusión se solicita definir, construir y presentar un Dashboard que permita medir los niveles de calidad de servicio, eficiencia y productividad del Call Center.
Para ello, se propone que definamos los KPIs adecuados para poder medir los objetivos propuestos, y definir nuevos niveles objetivos de manera de ofrecer esos niveles de SLA a terceras partes, o generar un nuevo servicio Premium para los clientes mas importantes del banco.

**Preguntas a responder**
- ¿Cuál es el nivel de servicio para los clientes Prioritarios? 
- ¿Damos un mejor servicio que a los clientes normales?
- ¿Qué volumen de llamadas atendemos? 
- ¿Cuáles son los cuellos de botella? ¿En qué días? ¿En qué bandas horarias?
- ¿Cómo es la eficiencia y productividad de nuestros agentes?
- ¿Hay clientes recurrentes en el uso del servicio?
- ¿Cuáles son los tipos de servicio más recurrentes?
- ¿Podemos estimar la dotación necesaria para cumplir con una calidad de servicio determinada?  Ejemplo: si quiero que mi tiempo promedio de espera sea menor a 60 segundos?
### Para comenzar, se hace la importación de librerías y módulos. Inicialmente, trabajaremos con pandas y csv:
```
import pandas as pd
import csv
```
## Se importa el CSV que contiene los datos que serán limpiados y posteriormente analizados:
```
df = pd.read_csv('Call_Center_1999_DataSet.csv', delimiter=';', encoding='UTF-8') 
```
## Reporte de incidencias relacionadas con la primera vista de los datos y acciones tomadas para asegurar la integridad y limpieza de los datos:
- Se verifica la existencia de valores nulos y filas vacías que, de eliminarse, no perjudican la calidad de los datos, por lo que se procede a su eliminación.
```
df.dropna(inplace=True)
```
- Se verifica la existencia de una columna (startdate) que no contiene valores y no está descrita en el diccionario de datos, por lo que se procede a su eliminación.
```
df= df.drop('startdate', axis=1)
```
- Se identifica el valor "AA" dentro de la columna "Type" que no corresponde a un tipo de actividad indicada en el diccionario de datos, por lo que se procede a la eliminación de los valores "AA" 
```
df.drop(df[df['type']=='AA'].index,inplace=True)
```
- Se identifican outliers en los datos identificados como "Phantom" en la columna "Outcome". De acuerdo al diccionario de datos, el código "Phantom" marca datos de llamada que deben ser ignorados, por lo que se procede a su eliminación. 
```
df.drop(df[df['outcome']=='PHANTOM'].index,inplace=True)
```
- Se exporta el CSV que ha sido limpiado.
```
df.to_csv('callcentercorregido.csv', index=False, sep= ';')
```
### Análisis Exploratorio de Datos (EDA)
- Se observan los datos, tipo de datos y un ejemplo de su contenido.
```
df.head(3)
```
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>vru.line</th>
      <th>call_id</th>
      <th>customer_id</th>
      <th>priority</th>
      <th>type</th>
      <th>date</th>
      <th>vru_entry</th>
      <th>vru_exit</th>
      <th>vru_time</th>
      <th>q_start</th>
      <th>q_exit</th>
      <th>q_time</th>
      <th>outcome</th>
      <th>ser_start</th>
      <th>ser_exit</th>
      <th>ser_time</th>
      <th>server</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AA0101</td>
      <td>33116</td>
      <td>9664491.0</td>
      <td>2</td>
      <td>PS</td>
      <td>1999-01-01</td>
      <td>0:00:31</td>
      <td>0:00:36</td>
      <td>5</td>
      <td>0:00:36</td>
      <td>0:03:09</td>
      <td>153</td>
      <td>HANG</td>
      <td>0:00:00</td>
      <td>0:00:00</td>
      <td>0</td>
      <td>NO_SERVER</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AA0101</td>
      <td>33117</td>
      <td>0.0</td>
      <td>0</td>
      <td>PS</td>
      <td>1999-01-01</td>
      <td>0:34:12</td>
      <td>0:34:23</td>
      <td>11</td>
      <td>0:00:00</td>
      <td>0:00:00</td>
      <td>0</td>
      <td>HANG</td>
      <td>0:00:00</td>
      <td>0:00:00</td>
      <td>0</td>
      <td>NO_SERVER</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AA0101</td>
      <td>33118</td>
      <td>27997683.0</td>
      <td>2</td>
      <td>PS</td>
      <td>1999-01-01</td>
      <td>6:55:20</td>
      <td>6:55:26</td>
      <td>6</td>
      <td>6:55:26</td>
      <td>6:55:43</td>
      <td>17</td>
      <td>AGENT</td>
      <td>6:55:43</td>
      <td>6:56:37</td>
      <td>54</td>
      <td>MICHAL</td>
    </tr>
  </tbody>
</table>
</div>

- Se identifican las estadísticas descriptivas relacionadas con la columna que contiene duración de la llamada en VRU, la que contiene la duración de la llamada en cola y la que contiene la duración de la atención del agente telefónico.
```
(df ['vru_time'].describe())
```
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>vru_time</th>
      <th>q_time</th>
      <th>ser_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>54022.000000</td>
      <td>54022.000000</td>
      <td>54022.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>9.954259</td>
      <td>52.952612</td>
      <td>147.958887</td>
    </tr>
    <tr>
      <th>std</th>
      <td>32.911385</td>
      <td>87.943279</td>
      <td>220.784324</td>
    </tr>
    <tr>
      <th>min</th>
      <td>-334.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>6.000000</td>
      <td>0.000000</td>
      <td>13.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>6.000000</td>
      <td>15.000000</td>
      <td>84.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>10.000000</td>
      <td>71.000000</td>
      <td>185.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3639.000000</td>
      <td>3164.000000</td>
      <td>6454.000000</td>
    </tr>
  </tbody>
</table>
</div>

- Ya que se identifican valores negativos en la columna VRU_time, indica que existen valores inválidos en esta tabla. Por ese motivo, se elimina la columna y se calcula restando VRU_entry a VRU_exit.

- Para lograrlo, se debe cambiar el delimitador de los valores de la columna de ':' a '.'
```
df['vru_exit'] = df['vru_exit'].str.replace(':','.')
df['vru_entry'] = df['vru_entry'].str.replace(':','.')
df['vru_exit'] = df['vru_exit'].str.replace(r'\.', '', regex=True).astype(int)
df['vru_entry'] = df['vru_entry'].str.replace(r'\.', '', regex=True).astype(int)
```
- Luego se castea el tipo de valor de la columna de string a int. 
```
df['vru_exit']= df['vru_exit'].astype(int)
df['vru_entry']= df['vru_entry'].astype(int)
```
- Se calcula la columna "vru_time" restando los valores de "vru_exit" y "vru_entry". Esta columna tiene los valores en segundos.
```
df['vru_time']= df['vru_exit']-df['vru_entry']
```
- Se observan los datos, tipo de datos y un ejemplo de su contenido nuevamente.
```
df.head
```
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>vru.line</th>
      <th>call_id</th>
      <th>customer_id</th>
      <th>priority</th>
      <th>type</th>
      <th>date</th>
      <th>vru_entry</th>
      <th>vru_exit</th>
      <th>q_start</th>
      <th>q_exit</th>
      <th>q_time</th>
      <th>outcome</th>
      <th>ser_start</th>
      <th>ser_exit</th>
      <th>ser_time</th>
      <th>server</th>
      <th>vru_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AA0101</td>
      <td>33116</td>
      <td>9664491.0</td>
      <td>2</td>
      <td>PS</td>
      <td>1999-01-01</td>
      <td>31</td>
      <td>36</td>
      <td>0:00:36</td>
      <td>0:03:09</td>
      <td>153</td>
      <td>HANG</td>
      <td>0:00:00</td>
      <td>0:00:00</td>
      <td>0</td>
      <td>NO_SERVER</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AA0101</td>
      <td>33117</td>
      <td>0.0</td>
      <td>0</td>
      <td>PS</td>
      <td>1999-01-01</td>
      <td>3412</td>
      <td>3423</td>
      <td>0:00:00</td>
      <td>0:00:00</td>
      <td>0</td>
      <td>HANG</td>
      <td>0:00:00</td>
      <td>0:00:00</td>
      <td>0</td>
      <td>NO_SERVER</td>
      <td>11</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AA0101</td>
      <td>33118</td>
      <td>27997683.0</td>
      <td>2</td>
      <td>PS</td>
      <td>1999-01-01</td>
      <td>65520</td>
      <td>65526</td>
      <td>6:55:26</td>
      <td>6:55:43</td>
      <td>17</td>
      <td>AGENT</td>
      <td>6:55:43</td>
      <td>6:56:37</td>
      <td>54</td>
      <td>MICHAL</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>

- Luego, para entender mejor la distribución y la manera en la que se comportan los datos, se grafican histogramas en subplot para poder ser comparados fácilmente.
- Crear una figura con 3 subplots verticales
```
fig, axes = plt.subplots(3, 1, figsize=(10, 12))
```
- Se definen los límites de los ejes, para que se pueda ver de cerca la distribución.
```
xlim_min, xlim_max = 0, 2000
ylim_min, ylim_max = 0, 6000
```
- Se crean los histogramas en cada subplot con los límites establecidos y se personaliza color.
```
axes[0].hist(df['vru_time'], bins=100, color= "#ef2b2d", edgecolor= 'black')
axes[0].set_xlabel('Duración en VRU')
axes[0].set_ylabel('Frecuencia')
axes[0].set_title('Histograma de Duración en VRU')
axes[0].set_xlim(xlim_min, xlim_max)
axes[0].set_ylim(ylim_min, ylim_max)

axes[1].hist(df['q_time'], bins=50, color="#66008c", edgecolor= 'black')
axes[1].set_xlabel('Duración en cola')
axes[1].set_ylabel('Frecuencia')
axes[1].set_title('Histograma de Duración en Cola')
axes[1].set_xlim(xlim_min, xlim_max)
axes[1].set_ylim(ylim_min, ylim_max)

axes[2].hist(df['ser_time'], bins=100,color="#00c6b2", edgecolor= 'black')
axes[2].set_xlabel('Duración en servicio')
axes[2].set_ylabel('Frecuencia')
axes[2].set_title('Histograma de Duración en servicio')
axes[2].set_xlim(xlim_min, xlim_max)
axes[2].set_ylim(ylim_min, ylim_max)

```
- Se ajusta el espaciado entre subplots y se muestra la figura.
```
plt.tight_layout()
plt.show()
```
![image](https://github.com/user-attachments/assets/2ca52444-e243-4dc4-90b7-97460d859946)


## Análisis de los Histogramas

Los histogramas presentados ofrecen una visión general de la distribución de las duraciones en tres etapas de un proceso: VRU, cola y servicio. Cada histograma nos revela características particulares sobre estos tiempos.

### Histograma de Duración en VRU:
La distribución de las duraciones en VRU muestra una clara asimetría positiva, lo que significa que la mayoría de los procesos tienen duraciones relativamente cortas, pero existe una proporción considerable de procesos que se extienden por periodos mucho más largos. Esta cola larga a la derecha sugiere la presencia de valores atípicos o eventos que causan demoras significativas en esta etapa.

### Histograma de Duración en Cola:
El histograma de las duraciones en cola presenta una forma similar al anterior, con una ligera asimetría positiva. Sin embargo, se observa que la concentración de datos se desplaza hacia valores más bajos en comparación con el histograma de VRU. Esto indica que, en promedio, los tiempos de espera en cola son menores que las duraciones totales en el sistema.

### Histograma de Duración en Servicio:
La distribución de las duraciones en servicio también muestra una asimetría positiva, aunque en menor medida que las otras dos. La mayoría de los procesos tienen duraciones de servicio relativamente cortas, pero al igual que en VRU, existe una cola de eventos con duraciones más largas. En términos generales, las duraciones en servicio se sitúan entre las duraciones en VRU y en cola.
