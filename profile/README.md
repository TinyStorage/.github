# Команда 2.5.

## Команда

- Гнипель Анна Владимировна: FullStack
- Папикян Сергей Седракович: TL + Mobile + Backend + SRE
- Пилипченко Степан Кириллович: Design + PM
- ~~Кривошеев Святослав Сергеевич~~

### Пространство и ТЗ

- [Notion](https://rhinestone-suede-4a3.notion.site/52f1207faddb4304b43b8391159b4692?v=1a5a2d62c7aa80ada4e5000c15cfb700&pvs=73)
- [ТЗ](../assets/TZ.pdf)

## Демо

- [Сайт](https://tiny-storage.online/app)
- [APK](https://github.com/TinyStorage/TinyStorage-Mobile/releases/tag/0.0.1)

### Репозитории

| Название                                                           | Описание                                                                 |
|--------------------------------------------------------------------|--------------------------------------------------------------------------|
| [📦 Backend](https://github.com/TinyStorage/TityStorage-Backend)   | ASP.NET API с OAuth2 авторизацией, хранением данных и логикой учёта      |
| [📱 Mobile](https://github.com/TinyStorage/TinyStorage-Mobile)     | Приложение на .NET MAUI для сканирования предметов и выполнения операций |
| [💻 Frontend](https://github.com/TinyStorage/TinyStorage-Frontend) | Веб-интерфейс для просмотра предметов и аудита                           |

### Видео

- [AdminMobile.mp4](https://drive.google.com/file/d/1TbBI2STGWXrby-NsTOqpUq5Wee71caVB/view?usp=drive_link)
- [AdminWeb.mp4](https://drive.google.com/file/d/1NB2W1eqKGuM1Twww17rSqR_mqLmzG_XZ/view?usp=drive_link)
- [LabworkerMobile.mp4](https://drive.google.com/file/d/1p9qCIypdbvjy0vCt6qmQa4NhOV_bSMaN/view?usp=drive_link)
- [LabworkerWeb.mp4](https://drive.google.com/file/d/1_ax6NxdgXN4CRUMWaxaIHAiZIIxToPpj/view?usp=drive_link)

## Учетные записи

| Логин           | Пароль          | Роли     |
|-----------------|-----------------|----------|
| itmo_labworker1 | itmo_labworker1 | Лаборант |
| itmo_labworker2 | itmo_labworker2 | Лаборант |
| itmo_admin      | itmo_admin      | Админ    |

## Тестовые данные

| ![](https://barcode.tec-it.com/barcode.ashx?data=46D32A7C-3852-43C7-8807-E203F056EDB6&code=MobileQRCode&translate-esc=on&eclevel=L) | ![](https://barcode.tec-it.com/barcode.ashx?data=46D32A7C-3852-43C7-8807-E203F056E111&code=MobileQRCode&translate-esc=on&eclevel=L) | ![](https://barcode.tec-it.com/barcode.ashx?data=46D32A7C-3852-43C7-1111-E203F056E111&code=MobileQRCode&translate-esc=on&eclevel=L) |
|-------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| Не зарегестрированный                                                                                                               | Не зарегестрированный                                                                                                               | Зарегестированный                                                                                                                   |


## Тех. артефакты

### Системная архитектура

```plantuml
@startuml

left to right direction

node "Yandex Cloud" as yc <<cloudSystem>> {
    node "Ubuntu Server" as ubuntu <<OS>> {
        node "Docker Environment" as docker <<executionEnvironment>> {
            node "Watchtower" as watchtower <<application>> {
                component "Watchtower" as watchtower_app <<application>> #white
            }

            frame "Docker Compose" as dev #line.dashed; {
                node "Frontend Container" as dev_frontend_container <<container>> {
                    component "Nginx" as dev_nginx <<web server>> #white    
                }
                node "Backend Container" as dev_backend_container <<container>> {
                    component "Kestrel" as dev_kestrel <<web server>> #white
                }
                node "Database Container" as dev_database_container <<container>> {
                    component "PostgreSQL" as dev_postgres <<DBMS>> #white
                }

                node "Keycloak Container" as dev_keycloak_container <<container>> {
                    component "Keycloak" as dev_keycloak <<application>> #white
                }

                node "Grafana Container" as dev_grafana_container <<container>> {
                    component "Grafana" as dev_grafana <<application>> #white
                }

                node "Prometheus Container" as dev_prometheus_container <<container>> {
                    component "Prometheus" as dev_prometheus <<application>> #white
                }

                node "Jaeger Container" as dev_jaeger_container <<container>> {
                    component "Jaeger" as dev_jaeger <<application>> #white
                }

                dev_kestrel -- dev_grafana : <<protocol>>\nHTTP
                dev_kestrel -- dev_prometheus : <<protocol>>\nHTTP
                dev_kestrel -- dev_jaeger : <<protocol>>\nHTTP
                dev_keycloak - dev_kestrel : <<protocol>>\nHTTPS
                dev_keycloak - dev_postgres : <<protocol>>\nODBC
                dev_postgres -- dev_kestrel : <<protocol>>\nODBC
                dev_nginx -- dev_kestrel : <<protocol>>\nHTTPS

                dev_frontend_container <. watchtower_app : <<watch>>
                dev_backend_container <. watchtower_app : <<watch>>
            }
        }
    } 
}

node "PC" as pc <<device>> {
    node "OS" as pd_os <<OS>> {
        component "Browser" as browser <<web browser>> #white
    }
}

node "Mobile" as mobile <<device>> {
    node "Android/IOS" as mobile_os <<OS>> {
        component ".NET MAUI App" as maui <<application>> #white
    }
}

browser -down- dev_nginx : <<protocol>>\nHTTPS
maui -down- dev_kestrel : <<protocol>>\nHTTPS
maui -[hidden]- dev_nginx

@enduml
```

### ERD

```plantuml
@startuml
left to right direction
hide circle

rectangle "tiny-storage" as storage <<schema>> #white;line:black;line.dashed;text:black {
entity "**items**" as items <<table>> {
*id: uuid <<PK>>
--
*name: text
*taken_by: integer
*created_at: timestamptz
*updated_at: timestamptz
}

    entity "**item_audits**" as item_audits <<table>> {
        *id: uuid <<PK>>
        --
        *item_id: uuid <<FK>>
        *edited_by: integer
        *property: text
        *value: text
        *created_at: timestamptz
    }

    items::id ||--|{ item_audits::item_id
}
@enduml
```
