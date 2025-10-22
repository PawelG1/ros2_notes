# URDF - Unified Robot Description Format - WIP !

## O URDF

plik .xml, opsiuje konstrukcje fizyczna robota, generalnie model 3D, informacje o przegubach, motorach?, masie elementów etc.

Dane z urdf sa uzywane w symulacjach, wizualizacjach

podobno do dobrego zrozumienia urdf trzeba ogarnac wczesniej tf2 - "transform"
[link do doku tf2](https://wiki.ros.org/tf2)  
ale j*bac to na razie

### podstawowe elementy URDF

#### links (ogniwa)

- czesci robota (powiedzmy takie prostopadłościany między "stawami")  
kazde ogniwo ma wiadomo mase, wymiary, geometrie

#### Joints (przeguby - "stawy")

- polaczenia miedzy ogniwami

##### rozrozniamy typy

**revolute** - przegub obrotowy u nas bedzie 6, 6stopni swobody z samego ramienia + 1def bedzie z chwytaka ale nie wiem jak tam on jest zbudowany

**fixed** - sztywne polaczenie np zamocowany gdzies do ramienia sensor

> kazdy joint ma minimalny/maksymalny kat obrotu (z,y,x), limit sily

### Transformacje

-opisuja gdzie jest kazde ogniwo wzgledem drugiego

> (ja rozumiem jako posów)

## przykład

![alt text](images/SC_Example.png)

```
base_link (podstawa - nieruchoma)
    ↓ (joint1 - fixed)
link1 (przegub 1)
    ↓ (joint2 - obrót)
link2 (przegub 2)
    ... i tak dalej ...
    ↓ (joint4 - obrót)
link4 (przegub)
    ↓ (joint5 - fixed)
link5 gripper_link (chwytak)
```

## tworzenie urdf

w katalogu
    src/sub_arm/urdf/

tworzymy plik urdf:
    touch sub_arm.urdf

### a w pliku zaczynamy od stworzenia robota:

```xml
<?xml version="1.0" ?>
<robot name="sub_arm">
<!--...-->
</robot>
```

nastepnie tworzymy w srodku pierwszy link - podstawe robota

```xml
<link name="base_link">
        <inertial>
            <mass value="">
            <origin zyx="" rpy="">
            <inertia ixx="" ixy ixz iyx iyy .. izz>
        </inertial>
        <visual>
            <origin zyx="" rpy="">
            <geometry>
                np. <cylinder radius="" length="">
            </geometry>
            <material>
                <color rgba"...">
            </material>
        </visual>
        <collision>
            <origin zyx="" rpy="">
            <geometry>
                to samo zazw. co wyzej. <cylinder ..>
            </geometry>
        </collision>
</link>

```

gdzie 
[inertial] - paramtery fizyczne (masa, inercja) 
>wartosci do tego rozumiem mamy sobie randomowe dobrac

[visual] - ofc  
|-> [origin] gdzie wzgledem linku? jest rysowany  
|-> [geometry] -ksztalt  

[collision] -geometria do detekcji kolizji -zazwyczaj to samo co w visual  

potem mozemy dodac joint zeby potem polaczyc nstp link itd itd  

joint:
```xml
  <joint name="joint1" type="revolute">
    <parent link="link2"/>
    <child link="link3"/>
    <origin xyz="0 0 0.3" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
    <limit lower="-1.57" upper="1.57" effort="100" velocity="1.0"/>
  </joint>
```
gdzie
[parent] i [child] to wiadomo ktore linki laczy  
[origin] gdzie jest ten przegub relatywnie do parent link  
[axis] wybieramy os obrotu np wybieramy os y to   
```xml
<axis xyz="0 1 0">
```
[limit] min/max kat, obciazenie, predkosc




# TIP: polecam zainstalowac rozszerzenie URDF Visualizer, 
potem wywolac ctrl+shift+p -> show preview urdf  
naprawde ulatwia to prace z tworzeniem modeli w urdf/xacro

## WAZNE: by rviz zadzialal z tym naszym urdf
musimy sobie dodac nawet pusty plik konfiguracyjny .rviz do src/sub_arm/config/

    touch display.rviz

wystarczy pusty na razie

Zauwazylem tez ze nalezy najpierw pouruchamiac wszytskie publishery itp i na samym koncu rviz2 i najlepiej w odzielnym terminalu, bo wystepuja bugi

### przykładowy model bardziej rozbudowany modedl
![alt text](<images/image copy 4.png>)

```xml
<?xml version="1.0"?>
<robot name="sub_arm">
    <!-- dummy root link to avoid KDL inertia issue -->
    <link name="world">
    </link>

    <!-- fixed joint from world to base_link -->
    <joint name="world_to_base" type="fixed">
        <parent link="world"/>
        <child link="base_link"/>
        <origin xyz="0 0 0" rpy="0 0 0"/>
    </joint>

    <!-- base link -->
    <link name="base_link">
        <inertial>
            <mass value="1.0"/><!-- Zmieniono z 0.0 na 1.0 -->
            <origin xyz="0.0 0.0 0.0"
             rpy="0.0 0.0 0.0"/> 
            <inertia ixx="0.1" ixy="0" ixz="0" iyy="0.1" iyz="0" izz="0.1"/>
        </inertial>
        <visual>
            <origin xyz="0 0 0" rpy="0 0 0"/>
            <geometry>
                <cylinder radius="0.10" length="0.05"/>
            </geometry>
            <material name="base_color">
                <color rgba="0.1 0.1 0.1 1.0"/>
            </material>
        </visual>

    </link>

    <!-- joint 1 - pierwszy przegub -->
    <!-- origin wyniesiony o 0.05 bo stoi na gorze base link -->
    <!-- obrot wokol osi z -->
    <joint name="joint1" type="revolute">
        <parent link="base_link"/>
        <child link="link1"/>
        <origin xyz="0 0 0.05" rpy = "0 0 0"/>
        <axis xyz="0 0 1"/>
        <limit lower="-3.14159" upper="3.14159" effort="100" velocity="1.0"/>
    </joint>

    <link name="link1">
        <inertial>
            <mass value="0.5"/>
            <origin xyz="0 0 0.0" rpy="0 0 0"/>
            <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.005"/>
        </inertial>
        <visual>
            <origin xyz="0 0 0.0" rpy="0 0 0"/>
            <geometry>
                <cylinder radius="0.02" length="0.05"/>
            </geometry>
            <material name="link1_color">
                <color rgba="0.9 0.9 0.9 1.0"/>
            </material>
        </visual>

    </link>

    <!-- joint 2 - pierwszy przegub -->
    <!-- origin wyniesiony o 0.05-->
    <!-- obrot wokol osi y -->
    <joint name="joint2" type="revolute">
        <parent link="link1"/>
        <child link="link2"/>
        <origin xyz="0 0 0.025" rpy = "0 0 0"/>
        <axis xyz="0 1 0"/>
        <limit lower="-3.14159" upper="3.14159" effort="100" velocity="1.0"/>
    </joint>

    <link name="link2">
        <inertial>
            <mass value="0.5"/>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.005"/>
        </inertial>
        <visual>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <geometry>
                <cylinder radius="0.02" length="0.05"/>    
            </geometry>
            <material name="link2_color">
                <color rgba="0.4 0.4 0.4 1.0"/>
            </material>
        </visual>
    </link>


    <!-- link 3-->
        <!-- obrot wokol osi y -->
    <joint name="joint3" type="revolute">
        <parent link="link2"/>
        <child link="link3"/>
        <origin xyz="0 0 0.05" rpy = "0 0 0"/>
        <axis xyz="0 1 0"/>
        <limit lower="-3.14159" upper="3.14159" effort="100" velocity="1.0"/>
    </joint>

    <link name="link3">
        <inertial>
            <mass value="0.5"/>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.005"/>
        </inertial>
        <visual>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <geometry>
                <cylinder radius="0.02" length="0.05"/>    
            </geometry>
            <material name="link2_color">
                <color rgba="0.9 0.9 0.9 1.0"/>
            </material>
        </visual>
    </link>

    <!-- link 4-->
        <!-- obrot wokol osi z -->
    <joint name="joint4" type="revolute">
        <parent link="link3"/>
        <child link="link4"/>
        <origin xyz="0 0 0.05" rpy = "0 0 0"/>
        <axis xyz="0 0 1"/>
        <limit lower="-3.14159" upper="3.14159" effort="100" velocity="1.0"/>
    </joint>

    <link name="link4">
        <inertial>
            <mass value="0.5"/>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.005"/>
        </inertial>
        <visual>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <geometry>
                <cylinder radius="0.02" length="0.05"/>    
            </geometry>
            <material name="link4_color">
                <color rgba="0.4 0.4 0.4 1.0"/>
            </material>
        </visual>
    </link>

        <!-- link 5-->
        <!-- obrot wokol osi y -->
    <joint name="joint5" type="revolute">
        <parent link="link4"/>
        <child link="link5"/>
        <origin xyz="0 0 0.05" rpy = "0 0 0"/>
        <axis xyz="0 1 0"/>
        <limit lower="-3.14159" upper="3.14159" effort="100" velocity="1.0"/>
    </joint>
    
    <link name="link5">
        <inertial>
            <mass value="0.5"/>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.005"/>
        </inertial>
        <visual>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <geometry>
                <cylinder radius="0.02" length="0.05"/>    
            </geometry>
            <material name="link4_color">
                <color rgba="0.9 0.9 0.9 1.0"/>
            </material>
        </visual>
    </link>

     <!-- link 6-->
        <!-- obrot wokol osi z -->
    <joint name="joint6" type="revolute">
        <parent link="link5"/>
        <child link="link6"/>
        <origin xyz="0 0 0.05" rpy = "0 0 0"/>
        <axis xyz="0 0 1"/>
        <limit lower="-3.14159" upper="3.14159" effort="100" velocity="1.0"/>
    </joint>
    
    <link name="link6">
        <inertial>
            <mass value="0.5"/>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.005"/>
        </inertial>
        <visual>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <geometry>
                <cylinder radius="0.02" length="0.05"/>    
            </geometry>
            <material name="link4_color">
                <color rgba="0.4 0.4 0.4 1.0"/>
            </material>
        </visual>
    </link>

     <!-- link 7 & 8- GRIPPER ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ -->
        <!-- obrot wokol osi y -->
    <joint name="joint7" type="fixed">
        <parent link="link6"/>
        <child link="link7"/>
        <origin xyz="0.02 0 0.05" rpy = "0 0 0"/>
        <axis xyz="0 0 1"/>
        <limit lower="-3.14159" upper="3.14159" effort="100" velocity="1.0"/>
    </joint>

    <link name="link7">
        <inertial>
            <mass value="0.5"/>
            <origin xyz="0 0 0.02" rpy="0 0 0"/>
            <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.005"/>
        </inertial>
        <visual>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <geometry>
                <box size="0.01 0.02 0.055"/>  
            </geometry>
            <material name="link4_color">
                <color rgba="0.1 0.1 0.2 1.0"/>
            </material>
        </visual>
    </link>

    <joint name="joint8" type="fixed">
        <parent link="link6"/>
        <child link="link8"/>
        <origin xyz="-0.02 0 0.05" rpy = "0 0 0"/>
        <axis xyz="0 0 1"/>
        <limit lower="-3.14159" upper="3.14159" effort="100" velocity="1.0"/>
    </joint>

    <link name="link8">
        <inertial>
            <mass value="0.5"/>
            <origin xyz="0 0 0.02" rpy="0 0 0"/>
            <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.005"/>
        </inertial>
        <visual>
            <origin xyz="0 0 0.025" rpy="0 0 0"/>
            <geometry>
                <box size="0.01 0.02 0.055"/>  
            </geometry>
            <material name="link4_color">
                <color rgba="0.1 0.1 0.2 1.0"/>
            </material>
        </visual>
    </link>

    <!-- GRIPPER_END~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ -->
</robot>



```


## Na razie mozna przejsc do notatek odn. ustawiania launchfile


