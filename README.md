# 🤖 DOFBOTpi5 — ROS 2 Jazzy
<img width="1070" height="1914" alt="image" src="https://github.com/user-attachments/assets/9f8cb2cc-e636-4050-85a9-600e4f50199a" />

Paquete de ROS 2 Jazzy para la visualización, descripción y análisis cinemático del brazo robótico **DOFBOTpi5** mediante URDF, RViz2 y TF2.

## 📋 Requisitos

- Ubuntu 24.04
- ROS 2 Jazzy
- Python 3
- RViz2
- Colcon
- robot_state_publisher
- joint_state_publisher_gui
- tf2_ros

Instalar las dependencias principales:

```bash
sudo apt update
sudo apt install ros-jazzy-rviz2 \
                 ros-jazzy-robot-state-publisher \
                 ros-jazzy-joint-state-publisher-gui \
                 ros-jazzy-tf2-ros \
                 python3-colcon-common-extensions
```

## 📁 Estructura del paquete

```text
dofbotpi5_jazzy/
├── CMakeLists.txt
├── package.xml
├── launch/
│   └── display.launch.py
├── config/
├── urdf/
│   └── dofbotpi5.urdf
├── meshes/
│   ├── Base_link.STL
│   ├── Link_1.STL
│   ├── Link_2.STL
│   ├── Link_3.STL
│   ├── Link_4.STL
│   ├── Link_5.STL
│   └── gripper.STL
├── rviz/
│   └── dofbot.rviz
└── worlds/
```

## 🚀 1. Compilar el paquete

Desde la raíz del workspace:

```bash
cd ~/ros2_ws

source /opt/ros/jazzy/setup.bash

colcon build --packages-select dofbotpi5_jazzy --symlink-install

source install/setup.bash
```

Verificar que ROS 2 reconoce el paquete:

```bash
ros2 pkg list | grep dofbotpi5_jazzy
```

## 🦾 2. Ejecutar DOFBOTpi5 en RViz2

Abrir una terminal y ejecutar:

```bash
cd ~/ros2_ws

source install/setup.bash

ros2 launch dofbotpi5_jazzy display.launch.py
```

Esto inicia:

- **robot_state_publisher:** publica las transformaciones del robot.
- **joint_state_publisher_gui:** permite modificar los ángulos de las articulaciones mediante controles deslizantes.
- **RViz2:** muestra el modelo tridimensional del DOFBOTpi5.

La configuración visual se carga desde `rviz/dofbot.rviz`, siempre que esté incluida en el archivo launch.

## 📐 3. Frame de la herramienta final (tool0)

El efector final del robot está definido mediante un frame adicional llamado `tool0`, conectado al último eslabón (`link_6`) mediante una articulación fija.

Código URDF:

```xml
<joint name="end_effector" type="fixed">
    <origin xyz="0.0 0.0 0.19153998" rpy="0.0 0.0 0.0"/>
    <parent link="link_6"/>
    <child link="tool0"/>
</joint>

<link name="tool0"/>
```

Esta configuración coloca `tool0` a **0.19153998 metros (191.54 mm)** sobre el eje Z positivo del marco `link_6`.

El frame `tool0` se mueve junto con el gripper al modificar las articulaciones.

## 📏 4. Medir desde la base hasta la herramienta final

Con el robot ejecutándose en RViz2, abrir una segunda terminal.

Cargar el entorno:

```bash
source /opt/ros/jazzy/setup.bash
source ~/ros2_ws/install/setup.bash
```

Consultar la transformación entre `base_link` y `tool0`:

```bash
ros2 run tf2_ros tf2_echo base_link tool0
```

El comando devuelve información de:

- Posición X, Y, Z en metros.
- Orientación mediante cuaterniones.
- Orientación en ángulos Roll, Pitch y Yaw.
- Transformación actualizada mientras se modifican las articulaciones.

### Medir desde world hasta tool0

```bash
ros2 run tf2_ros tf2_echo world tool0
```

Esto permite conocer la posición y orientación de la herramienta respecto al sistema global `world`.

Para solucionarlo, reemplaza toda esa sección de tu README.md por este código Markdown:

## 🔢 5. Matriz de transformación homogénea

La posición y orientación del efector final pueden representarse mediante la matriz homogénea:

$$
{}^{0}T_{tool0} =
\begin{bmatrix}
r_{11} & r_{12} & r_{13} & p_x \\
r_{21} & r_{22} & r_{23} & p_y \\
r_{31} & r_{32} & r_{33} & p_z \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

Donde:

- **R:** matriz de rotación de 3×3.
- **p:** vector de posición del efector final.
- **base_link:** marco de referencia 0.
- **tool0:** marco de la herramienta final.

La transformación puede obtenerse mediante:

```bash
ros2 run tf2_ros tf2_echo base_link tool0
```

- **R:** matriz de rotación de 3×3.
- **p:** vector de posición del efector final.
- **base_link:** marco de referencia 0, si se utiliza como base del modelo cinemático.
- **tool0:** marco de la herramienta final.

El comando `tf2_echo` proporciona la traslación y la orientación necesarias para construir esta matriz.

## 🔍 6. Visualizar los frames en RViz2

Para observar los sistemas de coordenadas:

1. Abrir RViz2 mediante `display.launch.py`.
2. Presionar **Add**.
3. Seleccionar **TF**.
4. Activar **Show Axes** y **Show Names**.
5. Localizar los frames `base_link` y `tool0`.

Esto permite visualizar cómo cambia la posición y orientación del efector final al modificar los ángulos del robot.

## 🛠️ 7. Validar el archivo URDF

Antes de ejecutar el robot, es recomendable comprobar su descripción:

```bash
check_urdf ~/ros2_ws/src/dofbotpi5_jazzy/urdf/dofbotpi5.urdf
```

Si el comando no está disponible:

```bash
sudo apt install liburdfdom-tools
```

## 📌 Comandos rápidos

**Terminal 1 — Ejecutar DOFBOTpi5**

```bash
cd ~/ros2_ws
source install/setup.bash
ros2 launch dofbotpi5_jazzy display.launch.py
```

**Terminal 2 — Transformación base → herramienta**

```bash
source ~/ros2_ws/install/setup.bash
ros2 run tf2_ros tf2_echo base_link tool0
```

**Terminal 2 — Transformación world → herramienta**

```bash
ros2 run tf2_ros tf2_echo world tool0
```

---
<img width="1456" height="586" alt="image" src="https://github.com/user-attachments/assets/abb7dbbb-e001-409e-a689-b113fb6562f3" />

## 📚 Referencias

- [ROS 2 Jazzy](https://docs.ros.org/en/jazzy/)
- [URDF — Robot Description](https://docs.ros.org/en/jazzy/Tutorials/Intermediate/URDF/URDF-Main.html)
- [RViz2](https://github.com/ros2/rviz)
- [TF2](https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Tf2/Tf2-Main.html)

---

**Proyecto:** DOFBOTpi5  
**ROS:** Jazzy Jalisco  
**Plataforma:** Ubuntu 24.04  
**Descripción:** Visualización y análisis cinemático de un brazo robótico mediante ROS 2.
