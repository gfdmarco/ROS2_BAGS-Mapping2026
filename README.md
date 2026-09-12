# ROS 2 Bag — Perception → Path Planning → Controle

Este bag registra e reproduz a comunicação entre a **Perception**, o **CarNode / PathPlanner** e a interface enviada para **Controle**.

A ideia é conseguir testar o pipeline depois, sem precisar estar com o carro ou com a Perception real rodando.

## Tópicos gravados

```text
/lidar_node/cones
/teste/route
/enable_control
```

### `/lidar_node/cones`

Tipo:

```text
std_msgs/msg/Float32MultiArray
```

É a saída da Perception e a entrada do `CarNode`.

Os valores vêm em grupos de 3:

```text
[lateral, frente, classe]
```

Convenção usada:

```text
classe 0 = cone azul
classe 1 = cone amarelo
```

O `CarNode` recebe esses cones e os passa para o `PathPlanner`.

### `/teste/route`

Tipo:

```text
geometry_msgs/msg/PoseStamped
```

É a saída do planejamento enviada para Controle.

Atualmente o `CarNode` publica um waypoint por vez, usando o primeiro waypoint real calculado pelo `PathPlanner`.

No código atual:

```python
x = frente
y = -lateral
```

com:

```text
frame_id = base_link
```

O sinal de `y` deve continuar de acordo com a convenção combinada com Controle.

### `/enable_control`

Tipo:

```text
std_msgs/msg/Bool
```

Controla se o `CarNode` publica ou não a rota:

```text
true  -> publica /teste/route
false -> para de publicar /teste/route
```

## Fluxo registrado

```text
Perception
    |
    v
/lidar_node/cones
    |
    v
CarNode
    |
    v
PathPlanner
    |
    v
/teste/route
    |
    v
Controle
```

## Pré-requisitos

É necessário ter:

- ROS 2 instalado;
- o ambiente ROS carregado;
- o pacote `autonomous_vehicle` compilado, caso o objetivo seja recalcular a rota;
- dependências ROS como `std_msgs` e `geometry_msgs`.

No workspace:

```bash
cd ~/Mapping2026/Pipeline
source install/setup.bash
```

Se ainda não estiver compilado:

```bash
cd ~/Mapping2026/Pipeline
colcon build --packages-select autonomous_vehicle
source install/setup.bash
```

## Verificar se o bag foi salvo corretamente

Na pasta onde o bag foi gravado:

```bash
ros2 bag info mock_realistic
```

O comando deve mostrar:

- duração;
- número de mensagens;
- tópicos;
- tipo de cada tópico;
- quantidade de mensagens por tópico.

Também é possível conferir os arquivos:

```bash
ls -lh mock_realistic
```

## Reproduzir o bag completo

```bash
ros2 bag play mock_realistic
```

Isso republica todos os tópicos gravados.

## Reproduzir em loop

```bash
ros2 bag play mock_realistic --loop
```

## Reproduzir mais devagar

Por exemplo, em metade da velocidade:

```bash
ros2 bag play mock_realistic -r 0.5
```

## Uso recomendado: recalcular a rota

Para testar novamente o `CarNode` ou o `PathPlanner`, não reproduza a saída antiga `/teste/route`.

Reproduza apenas a entrada da Perception:

```bash
ros2 bag play mock_realistic   --topics /lidar_node/cones
```

Em outro terminal, rode o `CarNode`:

```bash
cd ~/Mapping2026/Pipeline
source install/setup.bash
ros2 run autonomous_vehicle CarNode
```

O fluxo passa a ser:

```text
ROS 2 bag
    |
    v
/lidar_node/cones
    |
    v
CarNode atual
    |
    v
PathPlanner atual
    |
    v
/teste/route
```

Isso permite mudar o algoritmo e repetir o teste usando exatamente os mesmos dados de Perception.

## Ver a saída enviada para Controle

```bash
ros2 topic echo /teste/route
```

Devem aparecer mensagens `PoseStamped`.

## Conferir a conexão da Perception

```bash
ros2 topic info /lidar_node/cones -v
```

O `CarNode` deve aparecer como subscriber.

Também é possível visualizar os dados:

```bash
ros2 topic echo /lidar_node/cones
```

## Conferir a interface com Controle

```bash
ros2 topic info /teste/route -v
```

O `CarNode` deve aparecer como publisher.

Quando o nó real de Controle estiver rodando, ele deve aparecer como subscriber.

## Testar `/enable_control`

Desabilitar publicação da rota:

```bash
ros2 topic pub --once   /enable_control   std_msgs/msg/Bool   "{data: false}"
```

Habilitar novamente:

```bash
ros2 topic pub --once   /enable_control   std_msgs/msg/Bool   "{data: true}"
```

## Gravar um novo bag

```bash
ros2 bag record   -o mock_realistic   /lidar_node/cones   /teste/route   /enable_control
```

Finalize com:

```text
Ctrl+C
```

Depois confira:

```bash
ros2 bag info mock_realistic
```

## Versão futura com INS

Quando o SBG estiver publicando odometria real, vale gravar também:

```text
/imu/odometry
/tf
/tf_static
/mapping/lap_closed
```

Exemplo:

```bash
ros2 bag record   -o carro_completo   /lidar_node/cones   /teste/route   /enable_control   /imu/odometry   /tf   /tf_static   /mapping/lap_closed
```

## Resumo rápido

Reproduzir tudo:

```bash
ros2 bag play mock_realistic
```

Reproduzir só Perception:

```bash
ros2 bag play mock_realistic   --topics /lidar_node/cones
```

Rodar o `CarNode`:

```bash
cd ~/Mapping2026/Pipeline
source install/setup.bash
ros2 run autonomous_vehicle CarNode
```

Ver a saída para Controle:

```bash
ros2 topic echo /teste/route
```

Ver informações do bag:

```bash
ros2 bag info mock_realistic
```
