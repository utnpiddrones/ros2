# ROS2

Nodos de [ROS2 Foxy](https://docs.ros.org/en/foxy/index.html) para intercomunicación de la computadora de abordo y el sistema de control de [PX4](https://docs.px4.io/master/en/). Basado, en la imagen de Docker [utnpiddrones/ros2](https://hub.docker.com/repository/docker/utnpiddrones/ros2).

# Árbol de directorios del Docker
Dentro del Docker [utnpiddrones/rps](https://hub.docker.com/repository/docker/utnpiddrones/ros2), se construye un [entorno de trabajo colcon para ROS2](https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace.html?highlight=colcon), y se agregan al mismo dos nodos de PX4, siguiendo las instrucciones de [PX4-ROS2 Bridge](https://docs.px4.io/main/en/ros/ros2_comm.html).

* [px4_msgs](https://github.com/PX4/px4_msgs.git): Contiene las definiciones de todos los mensajes de ROS2 usados para comunicarse con PX4.
* [px4_ros_com](https://github.com/PX4/px4_ros_com.git): Contiene el agente de microRTPS que se encarga de manejar directamente la comunicación con el cliente iniciado sobre el drone.

```sh
# Directorio por defecto: /root/px4_ros_com_ros2

├──/entrypoint.sh
├──/root/
  └── px4_ros_com_ros2/
    ├── install/
    |   └── setup.bash
    ├── build/
    ├── log/
    └── src/
        ├── px4_msgs
        └── px4_ros_com
```

* **entrypoint.sh**: Script de inicialización. Todos los comandos puestos en este documento son ejecutados al iniciar el contenedor.

* **/root/px4_ros_com_ros2/**: Contiene el Colcon workspace. Incluye códigos fuente, archivos compilados, y entornos de ejecución.

  * **install/**: Contiene los archivos de entorno. Permiten ejecutar los comandos de ROS2, así como reconocer los paquetes.

    * **setup.bash**: Sourcear este archivo agrega al entorno el underlay de la instalación de ROS2, así como el overlay del actual colcon workspace.

  * **build/**: Contiene los archivos compilados.

  * **log/**: Contiene los logs de buildeo y ejecución.

  * **src/**: Contiene los distintos paquetes de ROS2.

    * **px4_msgs**: Paquete con mensajes de PX4.

    * **px4_ros_com**: Paquete con el agente de microRTPS, y ejemplos de ejecución.


# Uso

Primero, es necesario preparar el entorno de trabajo al sourcear el archivo de instalación:

```sh
$ source install/setup.bash
```

Para agregar un nodo propio de ROS2 "my_package", se debe crear un volumen con el nombre `/root/px4_ros_com_ros2/src/my_package`, y luego compilar el entorno de colcon con la siguiente línea:

```sh
$ colcon build --symlink-install --packages-skip px4_msgs px4_ros_com
```

Para comunicarse con el drone, es necesario iniciar en segundo plano el agente de microRTPS, y luego ejecutar el programa que se quiera.

```sh
# RPTS_PROTOCOL suele ser UDP o UART, y PX4_IP es la IP del contenedor.
$ micrortps_agent start -t ${RTPS_PROTOCOL} -i ${PX4_IP} > /dev/null &
```



## Modelo de servicio Docker Compose

```yaml
services:      
  ros2:
    image: "utnpiddrones/ros2:latest"

    # Terminal interactiva
    tty: true

    volumes:
      # Reemplaza el script de inicialización por defecto por uno propio
      - ${PWD}/entrypoint.sh:/entrypoint.sh
      # Agregar sus paquetes aquí:
      # - ${PWD}/my_package:/root/px4_ros_com_ros2


    # Nombre del contenedor al ejecutar.
    container_name: "ros2_cont"
```

## Modelo de archivo entrypoint.sh
```sh
#!/bin/bash

set -e

RTPS_PROTOCOL="UDP"
PX4_IP="127.0.0.1"

source /root/px4_ros_com_ros2/install/setup.bash 
# Este sleep se define para darle tiempo al cliente de PX4 a ejecutarse.
sleep 20
# En caso de tener un paquete propio, descomentar esta línea
# colcon build --symlink-install --packages-skip px4_msgs px4_ros_com
micrortps_agent start -t ${RTPS_PROTOCOL} -i ${PX4_IP} > /dev/null &
ros2 run px4_ros_com offboard_control

exec $@
```

# Tutorial de ROS1

Históricamente, se armó un [tutorial para el uso de ROS1](https://docs.google.com/document/d/1D_b0H5wiFdyh2-BmS-qTF1_-4NFg61R_zfV_gSrFMYg/edit?usp=sharing). Si bien ROS1 se dejó de usar en favor de ROS2, algunas funciones y conceptos todavía son relevantes.