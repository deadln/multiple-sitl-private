# Описание
Этот форк позволяет запускать симуляцию gazebo с двумя разными типасм моделями, работающими на прошивке PX4 и управляемыми с помощью MAVROS. Оригинальный [README](https://github.com/deadln/multiple-sitl/blob/develop/README).
# Установка
На вашем компьютере с ОС Ubuntu 20.04 должны быть установлены git (sudo apt install git) и [ROS Noetic](http://wiki.ros.org/noetic/Installation/Ubuntu).
Скачайте исходники PX4 с тегом vX.Y.Z и установите модуль sitl_gazebo
```
 git clone --branch vX.Y.Z https://github.com/PX4/PX4-Autopilot.git
 mv PX4-Autopilot Firmware
 cd Firmware
 git submodule update --init Tools/sitl_gazebo
```
Загрузите multiple-sitl с веткой multiple-models
```
cd Tools
git clone --branch multiple-models https://github.com/deadln/multiple-sitl.git
cd multiple-sitl/install
```
Установить окружение и симуляцию
```
./all.sh
cd ..
```
# Запуск симуляции
Для того чтобы запустить симуляцию с двумя моделями выполните:
```
./start_2models.rb -n количество_первых_моделей --gazebo_model первая_модель -n2 количество_вторых_моделей --gazebo_model2 вторая_модель
```
Примечание: для того чтобы в симуляции был приемлемый Real Time Factor рекомендуется загружать первой моделью менее стабильную и оптимизированную. Например, запустить симуляцию с одним ровером и двумя коптерами:
```
./start_2models.rb -n 1 --gazebo_model r1_rover -n2 2 --gazebo_model2 iris
```
Чтобы остановить симуляцию выполните:
```
./kill_sitl.sh
```
# Программное взаимодействие
Управление моделями осуществляется с помощью пакета MAVROS, передающего в симуляцию MAVLINK сообщения. Для каждой модели создаётся MAVROS-нода с именами:
* Для первой модели: mavros[1..n]
* Для второй модели: mavros[n+1..n+n2-1]
# Облёт поля
Для запуска симуляции облёта и объезда поля выполните команду:
```
./start_2models.rb -n 1 --gazebo_model r1_rover -n2 2 --gazebo_model2 iris --debug 10hect.world
```
После того как мир и модели загрузятся, выполнить:
```
python3 field_flyover.py
```
и ввести требуемые данные.
## Важно
Плагин определения местоположения модели r1_rover работает крайне нестабильно, и на поворотах он может перестать адекватно работать. Из-за этого скрипт облёта поля использует для ориентации ровера в пространстве данные о положении модели из самого gazebo. Однако через некоторое время после того как плагин перестаёт нормально работать ровер может остановиться или резко сбиться с траектории. Чтобы этого избежать, выполняйте следующие действия:
1) Всегда запускайте симуляцию с ровером с параметром --debug чтобы иметь доступ к консоли PX4
2) Найдите консоль ровера. Если вы запускали симуляцию по примеру выше, то консоль будет называться <b>px4-1</b>
3) Если ровер остановился или резко поехал в случайную сторону, пропишите в консоли эти команды:
```
ekf2 stop
ekf2 start
```
После этого ровер вернётся к своему прежнему маршруту
