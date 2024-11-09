### Вертуальные машины и Операциоенные системы


| Имя устройства | Процессор | Оперативная память | Жёский диск |     Операционная Система   |
| :------------: | :-------: |  :---------------: | :---------: |       :-------------:      |
| ISP            |  2 (ГЦ)   |        4 (ГБ)      |   8 (ГБ)    |            Eltex           |
| HQ-RTR         |  2 (ГЦ)   |        4 (ГБ)      |   6 (ГБ)    |          EcoRouter         |
| BR-RTR         |  2 (ГЦ)   |        4 (ГБ)      |   8 (ГБ)    |            Eltex           |
| HQ-SRV         |  2 (ГЦ)   |        4 (ГБ)      |  35 (ГБ)    |      ALT Linux Server      |
| BR-SRV         |  2 (ГЦ)   |        4 (ГБ)      |  35 (ГБ)    |      ALT Linux Server      |
| HQ-CLI         |  2 (ГЦ)   |        4 (ГБ)      |  35 (ГБ)    |    ALT Linux WorkStation   |\



## Создание EcoRouter

Выбираем способ устрановки
![Снимок экрана (16)](https://github.com/user-attachments/assets/da14de94-a11c-4c27-936e-a276c464757d)

Добовляем все файлы в поле
![Снимок экрана (16)](https://github.com/user-attachments/assets/33036248-ba78-42b5-8a5c-353c7205b6d6)

Нажимаем далее устанавливаем адаптеры и заканчиваем нажимая на `FINISH`

Ждём полной установки и запускаем её.

## Настройка vESR (Eltex)

выбираем операционную систему `Linux` и версию `Other (64-bit)`
![image](https://github.com/user-attachments/assets/eabec62d-339c-4b88-8393-1a25f4aff5fe)

Заходим в настройку жёсткого диска и меняем Controller Location и меняем на `IDE controller 0` -> `Master`
![Снимок экрана (17)](https://github.com/user-attachments/assets/48967970-8627-4a79-869f-ea547d07cba6)

Заканчиваем и запускаем.
После этого открывается окно установки. Нажимаем `Ок`
![image](https://github.com/user-attachments/assets/0d548c64-7f71-4a51-b12f-a4708f473e08)

После нажимаем на пробел чтобы выбрать диск установки и продолжаем нажимая `Ок`
![image](https://github.com/user-attachments/assets/834b0f0c-a129-4548-a964-2e38b2a95654)

Заканчиваем нажимая `Yes`
![image](https://github.com/user-attachments/assets/4acb0e3b-2d70-4523-a54f-5425544a56de)

После установки его нужно перезагрузить командой `reboot`
![image](https://github.com/user-attachments/assets/fddab497-806d-4784-b748-b0fe5454c8b9)

После запуска входим:<br>
 Login - `admin`<br>
 Password - `password`<br>
 И настраиваем новый пароль 
 ```
password P@ssw0rd
 ```
 Сохраняем прописывая `commit` и `confirm`<br>
