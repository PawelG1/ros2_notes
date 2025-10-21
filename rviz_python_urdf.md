# praca z urdf w publisher node dla rviz
generalnie zalecam zapoznac sie z [rviz_python_markers.md](rviz_python_markers.md)
bo nie chce mi sie pisac po 2 razy tego samego, wiec to raczej kontynuacja tamtych notatek

### Zakładam, że na ten moment:

* masz workspace ~/ros2_ws z pakietem sub_arm (typ ament_python).
* skrypt my_visu.py publikujacy marker i TF 

* venv jest skonfigurowane z --system-site-packages.

* RViz2 wyświetla czerwoną kulkę, a base_link jest dostępny jako frame.

---
## doinstalujemy dependencies
    sudo apt install ros-jazzy-robot-state-publisher ros-jazzy-joint-state-publisher ros-jazzy-joint-state-publisher-gui ros-jazzy-xacro

## dodajemy URDF do katalogu urdf
opisane w [urdf.md](urdf.md)

> opcjonalnie mozna uzyc Xacro by uproscic prace z xml ale idk jeszcze jak

sprawdz czy urdf jest GIT:  
    check_urdf ~/ros2_ws/src/sub_arm/urdf/sub_arm.urdf

wynik:  
```bash
(venv) mecharolnik@lenovo:~/ros2_ws/src/sub_arm/urdf$ check_urdf ~/ros2_ws/src/sub_arm/urdf/sub_arm.urdf
robot name is: sub_arm
---------- Successfully Parsed XML ---------------
root Link: world has 1 child(ren)
    child(1):  base_link
        child(1):  link1
```

## dodajemy dependencies do package.xml
```xml
<depend>robot_state_publisher</depend>
<depend>joint_state_publisher</depend>
<depend>joint_state_publisher_gui</depend>

```
~~<depend>xacro</depend>~~

## upewnij sie ze w setup.py sa podpiete urdf, rviz i ew. launch:  

data_files:
```python
   # URDF
    (os.path.join('share', package_name, 'urdf'), glob('urdf/*.urdf')),
    # Launch files
    (os.path.join('share', package_name, 'launch'), glob('launch/*.launch.py')),
    # RViz config (we keep both rviz/ and config/ patterns; actual file is in config/)
    (os.path.join('share', package_name, 'rviz'), glob('rviz/*.rviz')),
    (os.path.join('share', package_name, 'config'), glob('config/*')),

```

oraz w entry_points:
```python
entry_points={
        'console_scripts': [
            'my_visu = sub_arm.my_visu:main',
        ],
    },
```

## launch file:

```python


```