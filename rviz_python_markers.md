# tworzenie vizualizacji markera w rviz w paczce ros2 type python

## konfiguracja pakiety pod vizualizacje

przechdzimy do ros2_ws  
aktywujemy venv  
    source bin/activate

oznaczamy venv jako ignorowany przez colcon:
    touch ~/ros2_ws/venv/COLCON_IGNORE


## przygotowowanie venv

opisane w 
[python_venv.md](./python_venv.md)

## publikowanie transformacji TF, by frame byly widoczne

utworz plik np my_visu.py w katalogu sub_arm/sub_arm/my_visu.py


>wiele z kodu ponizej jest self explanatory, kod pochodzi od groka, jest prosty wiec vibe coded ujdzie

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from visualization_msgs.msg import Marker
from geometry_msgs.msg import Point, TransformStamped
from tf2_ros import TransformBroadcaster

class MyVisuNode(Node):
    def __init__(self):
        super().__init__('my_visu_node')
        self.publisher_ = self.create_publisher(Marker, 'visualization_marker', 10)
        self.tf_broadcaster = TransformBroadcaster(self)
        self.timer = self.create_timer(1.0, self.timer_callback)
        self.get_logger().info('MyVisuNode has been started')

    def timer_callback(self):
        # Publikuj marker
        marker = Marker()
        marker.header.frame_id = "base_link"
        marker.header.stamp = self.get_clock().now().to_msg()
        marker.ns = "my_namespace"
        marker.id = 0
        marker.type = Marker.SPHERE
        marker.action = Marker.ADD
        marker.pose.position = Point(x=0.0, y=0.0, z=0.0)  # Centrum układu
        marker.pose.orientation.w = 1.0
        marker.scale.x = 0.5
        marker.scale.y = 0.5
        marker.scale.z = 0.5
        marker.color.a = 1.0
        marker.color.r = 1.0
        marker.color.g = 0.0
        marker.color.b = 0.0
        self.publisher_.publish(marker)

        # Publikuj transformację TF: map -> base_link
        t = TransformStamped()
        t.header.stamp = self.get_clock().now().to_msg()
        t.header.frame_id = "map"
        t.child_frame_id = "base_link"
        t.transform.translation.x = 0.0
        t.transform.translation.y = 0.0
        t.transform.translation.z = 0.0
        t.transform.rotation.w = 1.0
        t.transform.rotation.x = 0.0
        t.transform.rotation.y = 0.0
        t.transform.rotation.z = 0.0
        self.tf_broadcaster.sendTransform(t)
        self.get_logger().info('Published marker and TF')

def main(args=None):
    rclpy.init(args=args)
    node = MyVisuNode()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

> uruchamiamy skrypt node publishera topica ```/visualization_marker```, inicjalizuje ```TransformBroadcaster``` dla publikacji transformacji TF

co 1sek wywoluje callback i publikuje martker i tf

marker u nas to taka czerwona kulka, na pozycji 0 0 0 w frame base_link

>transformacja TF - jak rozumiem pozwala rvizowi narysowac kulke w odpowiednim ukladzie odniesienia

```bash
(venv)mecharolnik@lenovo:~/ros2_ws/src$ cd sub_arm
(venv)mecharolnik@lenovo:~/ros2_ws/src/sub_arm$ cd sub_arm
(venv)mecharolnik@lenovo:~/ros2_ws/src/sub_arm/sub_arm$ ls
__init__.py  my_visu.py
(venv)mecharolnik@lenovo:~/ros2_ws/src/sub_arm/
(venv)mecharolnik@lenovo:~/ros2_ws/src/sub_arm/sub_arm$ python my_visu.py
```

### kod juz dziala, ale trzeba skonfigurowac pakiet by dalo sie to odpalic przez ros2

? ustawienie uprawnien:

    chmod +x ~/ros2_ws/src/my_visu_pkg/my_visu_pkg/my_visu.py

#### 1. edytujemy setup.py w naszym package

dodajemy nowy entry_point:
```python
entry_points={
    'console_scripts': [
        'my_visu = sub_arm.my_visu:main',
    ],
},
```
to pozwoli na uruchomienie skryptu przez ros2 run sub_arm my_visu


#### 2. dodajemy potrzebne dependencies do package.xml

```xml
  <depend>rclpy</depend>
  <depend>visualization_msgs</depend>
  <depend>tf2_ros</depend>
  <depend>std_msgs</depend>
  <depend>geometry_msgs</depend>
```

#### 3. ofc przebuduj package
    cd ~/ros2_ws  
    colcon build --packages-select sub_arm

#### 4. zsourceuj xd install
    source install/setup.bash

#### odpal i licz ze dziala
    ros2 run sub_arm my_visu

## ustawienie rviz2 by bylo cos widac xd

teraz gdy nasz marker publisher juz dziala mozemy uruchomic rviz w innym terminalu:

     ros2 run rviz2 rviz2

![ukazuje nam sie okno rviz](<images/image copy.png>)

dodajemy display:
![alt text](<images/image copy 2.png>)

wybieramy by topic:
![alt text](<images/image copy 3.png>)

i nastepnie wybieramy marker, zatwierdzamy "OK"

po tym powinna nam sie ukazac juz czerwona kulka - nasz marker

![alt text](<images/image copy 5.png>)
>jesli by cos nie dzialalo nalezy sie upewnic, ze mamy wybrane fixed_frame = [nazwa naszego ukladu odniesienia - u mnie base_link]

## teraz gdy cokolwiek nam dziala, mozemy zaczac rzezbic - zabawa markerkami 

mozemy dodac ruch markera

```python
..
self.count

def timer_callback(self):
    self.count += 0.1
    marker.pose.position = Point(x=math.sin(self.count), y=math.cos(self.count), z=0.0)
..
```

albo wincyj markerow (rozne id i ns - namespace?)

```python
marker2 = Marker()
marker2.header.frame_id = "base_link"
marker2.header.stamp = self.get_clock().now().to_msg()
marker2.ns = "my_namespace"
marker2.id = 1
marker2.type = Marker.CUBE
marker2.action = Marker.ADD
marker2.pose.position = Point(x=1.0, y=1.0, z=0.0)
marker2.pose.orientation.w = 1.0
marker2.scale.x = 0.5
marker2.scale.y = 0.5
marker2.scale.z = 0.5
marker2.color.a = 1.0
marker2.color.r = 0.0
marker2.color.g = 1.0
marker2.color.b = 0.0
self.publisher_.publish(marker2)
```

### mozna dodac subskrypcje do jakis danych z innego node np moze z IMU i na podstawie danych od niego aktualizowac visu:

```python 
self.subscription = self.create_subscription(
    geometry_msgs.msg.PoseStamped,
    '/robot_pose',
    self.pose_callback,
    10)
def pose_callback(self, msg):
    self.get_logger().info(f'Received pose: {msg.pose.position.x}')
    marker.pose.position = Point(
        x=msg.pose.position.x,
        y=msg.pose.position.y,
        z=msg.pose.position.z)
```

## praca z urdf
[rviz_python_urdf.md](rviz_python_urdf.md)

