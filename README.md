1-2. По первым двум пунктам согласен, можно сделать как Вы указали.  
3. По третьему пункту сообщаю, что в этом ресурсе я использую команду `yc managed-kubernetes cluster get-credentials master --external` для подключения к кластеру Яндекса. Вот ссылка https://yandex.cloud/ru/docs/managed-kubernetes/operations/connect/ . Соответственно файл создается на локальном компьютере, что позволяет использовать helm в terraform без указания провайдера, а также использовать команду kubectl.  
4. В разделе Создание тестового приложения было указано:
```
Ожидаемый результат:
Git репозиторий с тестовым приложением и Dockerfile.
Регистри с собранным docker image. В качестве регистри может быть DockerHub или Yandex Container Registry, созданный также с помощью terraform.
```
После создания регистри собирался образ и размещался в регистри согласно заданию, но при разборе ресурсов выдавало ошибку, что регистри не может быть удален, т.к. он содержит образ, соответственно добавил, чтобы образ тоже удалялся.  
5. Особого смысла создавать отдельные диски не было, просто экспериментировал, на случай, если вдруг понадобится для кластера дополнительное место.  
6.  Указал в пункте 4.  
7.  Можно сделать через команду helm install --name kube-prometheus --namespace=monitoring helm/kube-prometheus и при этом не нужно использовать провайдер. Также как в этом блоке сделано для ингресс контроллера. Просто использовал инструкция по установке kube-prometheus согласно ссылке в задании.  
8. Playbook запускается вручную, но для него автоматически создается ini-файл, а шаблон его находится в файле https://github.com/barankov-av/diplom/blob/main/terraform_ansible_kuberspray/terraform_work/templates/inventory_k8s.tpl 
Я бы потом сделал автоматический запуск плэйбука, но так как эта конструкция перестала быть рабочей, то я не стал терять на это время.  
9. Согласен, если бы не было бы это тестовое приложение, то так бы и сделал. В данном случае подумал, что так будет проще, т.к. используется для деплоя qbec, и для каждого namespace пришлось бы инициализировать отдельное окружение в qbec.  
10. Использую конструкцию:  
```
resource "null_resource" "install_patch" {

  provisioner "local-exec" {
    command = <<EOD
      if kubectl get nodes >/dev/null; then
        echo 'spec:' > external-ip.yaml
        echo '  externalIPs:' >> external-ip.yaml
        echo -n '  -  ' >> external-ip.yaml
        kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="ExternalIP")].address}' >> external-ip.yaml
        echo -n '\n  -  ' >> external-ip.yaml
        kubectl get nodes -o jsonpath='{.items[1].status.addresses[?(@.type=="ExternalIP")].address}' >> external-ip.yaml
        kubectl -n default svc ingress-nginx-controller --patch "$(cat external-ip.yaml)"
        echo -n '  -  ' >> external-ip.yaml
        kubectl get nodes -o jsonpath='{.items[3].status.addresses[?(@.type=="ExternalIP")].address}' >> external-ip.yaml
        kubectl -n default patch svc ingress-nginx-controller --patch "$(cat external-ip.yaml)"
      fi
    EOD
  }
}
```
Она создает файл с ip ВМ кластера и затем с помощью патча в ингресс-контроллер добавляются их ip, и тогда получаю доступ по ip любой ВМ с указанием порта ингресс контроллера.  
11. Не досмотрел, файл не сгенерирован для указания тега. Вместо этого нужно было указать `image: cr.yandex/crpsg85c77h25a1cb032/nginx-image:${tagValuenew}` Тогда генерируется файл с указанным тегом, без необходимости указания его вручную.  
