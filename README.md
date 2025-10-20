# ROS2 Notes

> notatki "robocze", mogą wprowadzać w błąd.. tutoriali nigdy nie pisałem xd
> 
> (mieszany polski i angielski :P)

---

## co tu znajdziesz

Szybkie notatki-poradniki z ROS2, testowane na Jazzy (Ubuntu 24.04).

### tematy:

- **[tworzenie_pakietow.md](./tworzenie_pakietow.md)** - podstawy tworzenia paczek ROS2 (ament_python/cmake)
- **[urdf.md](./urdf.md)** - URDF, czyli jak opisać robota w XML
- **[launch.md](./launch.md)** - launch files do uruchamiania kilku node'ów naraz
- **[python_venv.md](./python_venv.md)** - virtual env w kontekście ROS2 (jak nie zepsuć systemu)
- **[rviz_python_markers.md](./rviz_python_markers.md)** - wizualizacja markerów w RViz

---

## Quick start

### Setup środowiska

```bash
source /opt/ros/jazzy/setup.bash
cd ~/ros2_ws
source install/setup.bash
```

### Tworzenie paczki

```bash
cd ~/ros2_ws/src
ros2 pkg create nazwa_paczki --build-type ament_python --dependencies rclpy
```

### Build

```bash
cd ~/ros2_ws
colcon build --packages-select nazwa_paczki
source install/setup.bash
```

### Launch

```bash
ros2 launch nazwa_paczki plik.launch.py
```

---

## Struktura workspace

```
ros2_ws/
├── src/           # źródła paczek
├── build/         # pliki tymczasowe buildu
├── install/       # zainstalowane paczki
├── log/           # logi
└── venv/          # python virtual env (z COLCON_IGNORE!)
```

---

## Przydatne komendy

```bash
# lista paczek
ros2 pkg list

# info o paczce
ros2 pkg prefix nazwa_paczki

# lista node
ros2 node list

# topici
ros2 topic list
ros2 topic echo /topic_name

# czyszczenie buildu
rm -rf build/ install/ log/
```

---


**Uwaga:** To moje notatki. Nie gwarantuję że wszystko działa perfekcyjnie, ani ze podczas zabawy, przypadkowo nie usuniesz sobie wiekszosci aplikacji z systemu jak ja XD

