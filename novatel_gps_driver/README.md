# Localização global

*Driver's* para o sensor NovAtel SPAN IGM A1   (GPS+INS).
Adaptados de SwRI.

## Novos parâmetros

    Parâmetros de origem podem ser observados no [Github](https://github.com/swri-robotics/novatel_gps_driver)

```
    publish_default_messages:   -> publicação dos tópicos */fix* e */gps* 
    publish_novatelgnss_positions:   ->publica tópico com informação relativa ao posicionamento global baseado apenas no sistema GPS.
```



# Configuração do sensor
Deve ser realizado quando o sensor fornecer leituras estranhas.

Através de um programa de comunicação, por exemplo, *cutecom* ou *moserial*.

Deve ser utilizado o cabo USB(preto), e enviados os comandos seguintes: (o sensor tem de estar ligado)

```
FRESET
```

(espere até que os dois led fiquem verdes)

```
SERIALCONFIG COM1 230400 N 8 1 N ON
SERIALCONFIG COM2 230400 N 8 1 N ON
SERIALCONFIG COM3 230400 N 8 1 N ON

SETIMUORIENTATION 2
VEHICLEBODYROTATION 0.0 0.0 180.0 0.0 0.0 0.1
APPLYVEHICLEBODYROTATION enable
SETIMUTOANTOFFSET -0.80 -0.30 0.50 0.01 0.01 0.01

SETINSOFFSET 0.5 -1.3 0.5

ALIGNMENTMODE UNAIDED
SAVECONFIG
```

Em todos estes comandos o sensor deve responder *[OK]*

Estes parâmetros apenas são válidos para a instalação realizada por Pedro Bouça Nova 2018.

No caso de alterações no posicionamento do sensor ou antena, estes comandos devem ser revistos.  

Após este procedimento é aconselhável a realização de um alinhamento, sem desligar o sensor.  Isto é, este processo deve ser realizado num local em campo aberto e plano (se possível), e deve permitir o veículo percorrer em linha reta a 20/25 km/h durante 15/20s.   Por exemplo, parque de estacionamento ESSUA. 


# Utilização

Verificar o número dos *dev* das duas portas utilizadas, e introduzir valores no ficheiro  *novatel.launch*.

Cabo com conversor RS232-USB(branco) parâmetros relativos ao posicionamento.
Cabo USB(preto) parâmetros relativos aos dados inerciais. 

## Launch file
```
<?xml version="1.0"?>
<launch>
    <!-- Node novatel para comandos relativos a posição-->
    <node name="novatel_position" pkg="nodelet" type="nodelet" args="standalone novatel_gps_driver/novatel_gps_nodelet"><rosparam>
      connection_type: serial
      device: /dev/ttyUSB0
      publish_novatel_positions: true
      publish_novatelgnss_positions: false
      publish_imu_messages: false
      publish_nmea_messages: true
      publish_default_messages: true
      publish_diagnostics: true
      frame_id: /gps
  </rosparam>
  </node>
    <!-- Node novatel para comandos da unidade inercial -->
    <node name="novatel_imu" pkg="nodelet" type="nodelet" args="standalone novatel_gps_driver/novatel_gps_nodelet"><rosparam>
      connection_type: serial
      device: /dev/ttyUSB1
      publish_novatel_positions: false
      publish_novatelgnss_positions: false
      publish_imu_messages: true
      publish_nmea_messages: false
      publish_default_messages: false
      publish_diagnostics: false
      publish_novatel_velocity: true
      frame_id: /gps
  </rosparam></node>
    <!-- Transformacao geométrica com valores de gps ..... nas situações de paragem ou de baixa velocidade a componente Yaw perde-se ..(a imagem gira) -->
    <!-- <node pkg="swri_transform_util" type="gps_transform_publisher" name="gps_transform_publisher" output="screen"/> -->
    <!-- Transformacao geométrica com valores de gps+unidade inercial  ... Resolve o problema do gps_transform_publisher nas situções de paragem ou velocidade baixa  mas não está disponivel durante o processo de alinhamento .................. dá origem ao sistema de eixos "base_link_imu"   utiliza o topic /bestpos para obter a coordenada X, Y, Z no mapa e o tópico /inspva para obter o Yaw.-->

    <node pkg="swri_transform_util" type="imu_transform_publisher" name="imu_transform_publisher" output="screen"/>
   
  <node pkg="tf" type="static_transform_publisher" name="swri_transform" args="0 0 0 0 0 0 /world /map 100" />

    <!-- <node pkg="mapviz" type="mapviz" name="$(anon mapviz)" required="true" output="log"/> -->
    <node pkg="swri_transform_util" type="initialize_origin.py" name="initialize_origin" output="screen">
        <param name="local_xy_frame" value="/world"/>
        <param name="local_xy_origin" value="auto"/>
        <!-- "auto" setting will set the origin to the first gps fix that it recieves -->
        <remap from="gps" to="/gps/fix"/>
    </node>
</launch>
```

## Teste

Tópico com a leitura de posicionamento global. 
```
rostopic echo /bestpos
```


Tópico com os valores de roll, pitch e Yaw do veículo. 
```
rostopic echo /inspva
```
verificar a última variável *status*; deve ter o valor:

*status: ALIGNMENT_COMPLETE*

ou

*status: INS_SOLUTION_GOOD*

Em caso contrario é necessário relançar os driver's ou calibrar o sensor (desligar o sensor e seguir em linha reta a uma velocidade superior a 20km/h).

# Instalação

Através do github ATLASCAR.

Caso contrário os *driver's* não poderão ser lançados através de dois nodos para comunicar através de duas portas. 

## Autor
* **Pedro Bouça Nova** - *Dissertação de Mestrado - 2018* -




