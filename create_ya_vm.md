
## Создаем виртуальную машину
```bash
yc compute instance create \
  --name wb-pg-01 \
  --hostname wb-pg-01 \
  --zone ru-central1-b \
  --network-interface subnet-name=default-ru-central1-b,nat-ip-version=ipv4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-2204-lts,size=15G,type=network-ssd,auto-delete=true \
  --ssh-key ~/.ssh/id_ed25519.pub
```

## Проверяем статус ВМ
```bash
yc compute instance list
yc compute instance get wb-pg-01
yc compute instance get --full wb-pg-01
```

## Подключаемся к ВМ по SSH
```bash
ssh -i ~/.ssh/id_ed25519 yc-user@{one_to_one_nat from instance get}
```

## Подключаемся к ВМ через OS Login
```bash
yc compute ssh \
  --name wb-pg-01 \
  --folder-id b1ggc2f8als4gr23fiqr \
  --identity-file ~/.ssh/id_ed25519 \
  --login yc-user
```

## Подключаемся к ВМ чере OS Login
```bash
yc organization-manager organization list
yc organization-manager os-login profile list --organization-id bpfu7apvnbp9ikf0ub88
```


## Удаление ВМ
```bash
# yc compute instance stop wb-pg-01
# yc compute instance start wb-pg-01
# yc compute instance restart wb-pg-01
yc compute instance delete wb-pg-01
```
