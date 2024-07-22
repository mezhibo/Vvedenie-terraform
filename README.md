**Задание 1**

1) Перейдите в каталог src. Скачайте все необходимые зависимости, использованные в проекте.

2) Изучите файл .gitignore. В каком terraform-файле, согласно этому .gitignore, допустимо сохранить личную, секретную информацию?(логины,пароли,ключи,токены итд)

3) Выполните код проекта. Найдите в state-файле секретное содержимое созданного ресурса random_password, пришлите в качестве ответа конкретный ключ и его значение.

4) Раскомментируйте блок кода, примерно расположенный на строчках 29–42 файла main.tf. Выполните команду terraform validate. Объясните, в чём заключаются намеренно допущенные ошибки. Исправьте их.

5) Выполните код. В качестве ответа приложите: исправленный фрагмент кода и вывод команды docker ps.

6) Замените имя docker-контейнера в блоке кода на hello_world. Не перепутайте имя контейнера и имя образа. Мы всё ещё продолжаем использовать name = "nginx:latest". Выполните команду terraform apply -auto-approve. Объясните своими словами, в чём может быть опасность применения ключа -auto-approve. Догадайтесь или нагуглите зачем может пригодиться данный ключ? В качестве ответа дополнительно приложите вывод команды docker ps.

7) Уничтожьте созданные ресурсы с помощью terraform. Убедитесь, что все ресурсы удалены. Приложите содержимое файла terraform.tfstate.

8) Объясните, почему при этом не был удалён docker-образ nginx:latest. Ответ ОБЯЗАТЕЛЬНО НАЙДИТЕ В ПРЕДОСТАВЛЕННОМ КОДЕ, а затем ОБЯЗАТЕЛЬНО ПОДКРЕПИТЕ строчкой из документации terraform провайдера docker. (ищите в классификаторе resource docker_image )





**Решение 1**

![alt text](https://github.com/mezhibo/docker-compose/blob/01d7e387556b270ba6c00a9fc823b4e908750d3f/IMG/1.jpg)


1) Скачиваем файлы для домашнего задания (скрин сделан после инициалоизации, фаил .terraformrc перемещен в папку /home/ubuntu)

   ![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/3e3f4d803fc904bba4fada21e3cae02a0107edfb/IMG/3.jpg)


Приверим корректность установки Terraform

![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/3e3f4d803fc904bba4fada21e3cae02a0107edfb/IMG/1.jpg)


Теперь необходимо сделать инициализацию нашего каталога для дальнейшего выполнения задания. делаем в этйо директории terrafrom init

![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/3e3f4d803fc904bba4fada21e3cae02a0107edfb/IMG/2.jpg)



2) Согласно файлу .gitignore в файле терраформ personal.auto.tfvars допускается хранить секретные данные.

   ![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/3e854dd0d95703c0b43a8573959d44d12580f173/IMG/4.jpg)



3) После выполнения стейта получаем секретное значение random_password, оно лежит в файлике terraform.tfstate

![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/477f9bd4307d7b9b1e24f9ab0430267c01b85258/IMG/5.jpg)


4) Делаем terrafrom validate и видим ошибки

![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/ff3141003e416862df559a0ffbff516850a7cdb6/IMG/7.jpg)


- Идем в код и видим схожду первую ошибку на 24 строке, котору даже расширение для VS Code само подсвечивает, а именно 
у нас не указан тип ресурса для docker_image, ведь должно быть 2 значения

![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/ff3141003e416862df559a0ffbff516850a7cdb6/IMG/6.jpg)


 - На 29 строке видим цифру в названии, а так быть не должно, либо буква, либо нижнее подчеркивание

Исправил 2 ошибки что были, сделал validate, нашлась 3 ошибка на 31 строке 


![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/85ef0d1fad36f876d02210d1a080a0843a80a251/IMG/8.jpg)


Здесь видно что у нас название ресурса задано random_string, а там random_string_FAKE и еще resulT написан через большие буквы.

![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/cb3ef4cfbeb1ce1e8a49f5ec14fcd05c43b30581/IMG/9.jpg)


 - Исправляем ошибки и приводим код к правильному рабочему виду

'''

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string.result}"

  ports {
    internal = 80
    external = 9090
  }
}


'''

![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/c0f5c422f40e9beae0d08045831dfe9fa4316648/IMG/10.jpg)


5) Делаем terraform plan, и проверяем что все отработало


![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/519b90ea269286cc2d03362d394fa94257771377/IMG/11.jpg)



6) Меняем имя контейнера на новое

   ![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/1361d03f4c989e09f79196b35822535c7b28a2f5/IMG/13.jpg)


Запускаем terraform apply -auto-approve, и снова смотрим docker ps, видим что имя конетйнера у нас поменялось без нашего подверждения 


![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/1361d03f4c989e09f79196b35822535c7b28a2f5/IMG/14.jpg)

И сразу видно что terraform не спрашивая нас, сама пересоздала контейнер с новым именем.


Ключ -auto-approve автоматичсеки применяем новое описание в манифестах к сущесвующей конфигурации без потверждения. В практике такое лучше использовать либо в тестовых средах, либо во всяких Ci/Cd типо Jenkins для автоматического аппрува изменения конфигурации.


7) Для уничтожения всей конфигурации будем использовать terrafrom destroy

   ![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/8505e3f047791150b60a7175c2bae4955406f5f3/IMG/15.jpg)


   и теперь глянем внутрь файлика terraform.tfstate

   ![alt text](https://github.com/mezhibo/Vvedenie-terraform/blob/8505e3f047791150b60a7175c2bae4955406f5f3/IMG/16.jpg)


8) Докер образ nginx:lates не был удален, потому что
   потому что использовали параметр keep_locally = true при создании iamge.
   В документации написано:

 [Keep_locally (Boolean) If true, then the Docker image won't be deleted on destroy operation. If this is false, it will delete the image from the docker local storage on destroy operation](https://docs.comcloud.xyz/providers/kreuzwerker/docker/latest/docs/resources/image)




