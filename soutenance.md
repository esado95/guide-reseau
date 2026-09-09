# Тренаж защиты

Вопросы, которые задаёт комиссия на устной защите учебной работы
по построению сетевого стенда. Ответ дан по-французски — его можно
произнести как есть. Под каждым по-русски: что на самом деле
проверяют этим вопросом.

Сопроводительные материалы: отчёт POC и руководство
https://esado95.github.io/guide-reseau/

---

Тридцать пять вопросов, которые задаёт комиссия. Ответ по-французски — его можно произнести как есть. Под ним по-русски: что на самом деле проверяют этим вопросом.


Работай так: читаешь вопрос, отвечаешь вслух по-французски, и только потом раскрываешь. Если ответ совпал по смыслу — хорошо, дословно совпадать не обязательно. Если не нашёл слов — это и есть твоя слабая точка, отметь её.



**Структура любого ответа**


Три части, в этом порядке: **что сделал → почему именно так → чем доказано**. Третья часть отличает уверенный ответ от заученного: ты не просто утверждаешь, а называешь проверку, которая это подтверждает.





**Чего не делать**


Не начинай с команд. «J'ai tapé `ip routing`» — плохой ответ, он показывает память, а не понимание. Начинай с задачи: «Il fallait que les VLAN communiquent entre eux, donc…». Команда идёт в конце, как деталь.




## Общие вопросы об архитектуре



### Présentez votre infrastructure en deux minutes.


**Réponse.** Le réseau est segmenté en quatre VLAN : deux pour les clients, un pour le serveur DNS, un pour l'administration. Le routage entre ces VLAN est assuré par un commutateur de niveau 3, qui distribue aussi les adresses aux clients. Une liaison point à point relie ce commutateur au pare-feu pfSense, qui possède trois interfaces : le réseau de la salle, le réseau interne et une DMZ. Dans la DMZ se trouve un serveur Debian avec deux sites Apache : un site interne sur le port 80 et un site public sur le port 8080, publié vers l'extérieur par une redirection de port.


**По-русски.** Проверяют, держишь ли ты картину целиком. Отвечай сверху вниз: сегментация → маршрутизация → выход наружу → публикация. Не углубляйся в детали, их спросят отдельно.




### Pourquoi avoir segmenté le réseau en VLAN ?


**Réponse.** Pour trois raisons. D'abord isoler les groupes d'utilisateurs les uns des autres. Ensuite limiter le trafic de diffusion, qui reste confiné dans son VLAN. Enfin, et c'est le plus important, la segmentation est ce qui rend le filtrage possible : tant que tout le monde est dans le même segment, aucun pare-feu ne peut s'interposer.


**По-русски.** Слабый ответ — «для безопасности». Сильный называет механизм: без сегментации политику применить не к чему, трафик до firewall просто не доходит.




### Qui assure le routage entre les VLAN, et pourquoi ce choix ?


**Réponse.** Le commutateur de niveau 3. Chaque VLAN possède une interface virtuelle, le SVI, qui sert de passerelle par défaut aux machines de ce VLAN. Ce choix est le plus rapide : le routage est fait en matériel, au plus près des postes. Les alternatives seraient un routeur avec des sous-interfaces, ou le pare-feu lui-même — mais dans les deux cas tout le trafic inter-VLAN traverserait une seule liaison.


**По-русски.** Здесь ждут, что ты знаешь три способа и можешь сравнить. Назови все три и обоснуй выбор.




### Qu'est-ce qu'une DMZ et pourquoi le serveur web s'y trouve-t-il ?


**Réponse.** Une DMZ est un segment réservé aux services exposés vers l'extérieur. L'idée est qu'un serveur compromis ne donne pas accès au réseau interne. C'est pour cela qu'aucune règle n'autorise la DMZ à initier une connexion : le serveur répond aux demandes entrantes, mais ne peut rien joindre de lui-même.


**По-русски.** Проверяют, понимаешь ли ты смысл, а не название. Ключ — «сервер только отвечает, сам никуда не ходит».




### Expliquez votre plan d'adressage. Pourquoi des masques différents ?


**Réponse.** Le masque est choisi selon le nombre de machines attendues. Les réseaux clients sont en /24 parce qu'ils doivent accueillir beaucoup de postes. Le VLAN d'administration est en /28, soit quatorze adresses utilisables, ce qui suffit largement. Le VLAN du serveur DNS est en /29, six adresses. La liaison entre le pare-feu et le serveur web est en /30 : exactement deux adresses, c'est une liaison point à point.


**По-русски.** Могут попросить посчитать вслух. Помни: из общего числа адресов всегда вычитаются два — адрес сети и широковещательный.





## Коммутатор



### Quelle est la différence entre un port en mode access et un port en mode trunk ?


**Réponse.** Un port access appartient à un seul VLAN et transmet les trames sans étiquette : la machine connectée ignore l'existence des VLAN. Un port trunk transporte plusieurs VLAN sur le même lien physique, et chaque trame reçoit une étiquette 802.1Q qui indique son VLAN. On utilise un trunk vers un autre commutateur, un hyperviseur ou un point d'accès.


**По-русски.** Базовый вопрос, но на нём часто плывут. Обязательно назови тег 802.1Q — без него ответ неполный.




### Qu'est-ce qu'un SVI ?


**Réponse.** C'est une interface virtuelle de VLAN sur le commutateur. On lui attribue une adresse IP, et cette adresse devient la passerelle par défaut des machines du VLAN. C'est par les SVI que le commutateur de niveau 3 route le trafic d'un VLAN vers un autre.


**По-русски.** Односложно: виртуальный интерфейс VLAN с IP-адресом, он же шлюз для этого VLAN.




### Vous avez créé les SVI, mais rien ne passe entre les VLAN. Pourquoi ?


**Réponse.** Parce que la fonction de routage n'est pas activée par défaut sur le commutateur. Sans elle, les SVI existent, sont actifs, ont une adresse — mais le commutateur ne route rien. Le symptôme est caractéristique : chaque machine joint sa propre passerelle, mais aucune ne joint un autre VLAN.


**По-русски.** Любимый вопрос-ловушка. Главное — назвать симптом «свой шлюз да, чужой нет», это доказывает, что ты это видел, а не читал.




### Pourquoi le port vers le pare-feu n'est-il pas dans un VLAN ?


**Réponse.** Parce qu'il s'agit d'une liaison routée point à point. La commande `no switchport` transforme ce port en port de routeur : il quitte le monde des VLAN et reçoit sa propre adresse IP. C'est plus simple qu'un VLAN de transit, et cela évite d'étendre un domaine de diffusion jusqu'au pare-feu.


**По-русски.** Могут спросить «а можно было через VLAN?» Да, можно, но точка-точка проще и чище.




### Pourquoi avoir limité la liste des VLAN autorisés sur les trunks ?


**Réponse.** Pour que chaque équipement ne reçoive que les segments dont il a besoin. Le point d'accès reçoit les VLAN clients et le VLAN d'administration, l'hyperviseur reçoit ceux de ses machines virtuelles. Tout VLAN supplémentaire serait à la fois du trafic de diffusion inutile et un chemin possible vers un segment interdit.


**По-русски.** Ответ «чтобы было аккуратно» слабый. Назови обе причины: лишнее широковещание и лишняя дорожка.




### Qu'est-ce que le VLAN natif, et pourquoi l'avoir déplacé ?


**Réponse.** Le VLAN natif est celui dont les trames circulent sur le trunk sans étiquette. Par défaut c'est le VLAN 1. Je l'ai déplacé vers un VLAN dédié et inutilisé pour que toute trame non étiquetée tombe dans un segment vide. Cela neutralise l'attaque par double étiquetage et évite qu'un trafic imprévu se retrouve dans un réseau de production.


**По-русски.** Если знаешь термин «double tagging» — назови, это сразу поднимает уровень ответа.





## Адресация и имена



### Pourquoi le service DHCP est-il sur le commutateur et non sur le pare-feu ?


**Réponse.** Parce que la demande d'adresse est une trame de diffusion, et une diffusion ne sort pas de son VLAN. Le commutateur est déjà la passerelle de chaque VLAN, donc il se trouve à l'intérieur de chaque domaine de diffusion et reçoit la demande directement. C'était aussi une exigence du cahier des charges.


**По-русски.** Начни с механизма, а не с «так требовалось». Требование потом, как подтверждение.




### Et si le serveur DHCP était dans un autre VLAN ?


**Réponse.** Il faudrait activer le relais DHCP sur la passerelle de chaque VLAN client. Le relais transforme la demande de diffusion en une demande unicast adressée au serveur. Cela fonctionne, mais ajoute un maillon supplémentaire et donc un point de panne de plus.


**По-русски.** Проверка глубины. Термин по-французски — *relais DHCP*, у Cisco команда `ip helper-address`.




### Quels paramètres distribuez-vous en plus de l'adresse ?


**Réponse.** Le masque, la passerelle par défaut, l'adresse du serveur DNS et le suffixe DNS du domaine interne. J'exclus aussi les vingt premières adresses de chaque plage, réservées aux attributions fixes.


**По-русски.** Четыре параметра плюс исключения. Забыть суффикс — типовая ошибка, о которой спросят следующим вопросом.




### À quoi sert le suffixe DNS distribué par le DHCP ?


**Réponse.** Il permet aux clients d'utiliser des noms courts. Sans suffixe, la machine ne sait pas quoi ajouter à `web` et la résolution échoue, alors que `web.abu` fonctionne. C'est exactement le symptôme : le nom complet répond, le nom court non.


**По-русски.** Назови симптом — это показывает, что ты понимаешь, а не пересказываешь.




### À quoi servent les redirecteurs sur le serveur DNS ?


**Réponse.** Le serveur interne fait autorité uniquement sur la zone interne. Pour tout le reste, il transmet la question à des serveurs publics. Sans redirecteurs, les clients résoudraient les noms internes mais aucun nom d'Internet.


**По-русски.** Ключевое слово по-французски — *redirecteurs*. Симптом: внутренние имена работают, внешние нет.





## Маршрутизация и трансляция адресов



### Pourquoi avoir déclaré des routes statiques sur le pare-feu ?


**Réponse.** Le pare-feu ne connaît que ses réseaux directement connectés. Les quatre réseaux internes se trouvent derrière le commutateur de niveau 3, donc j'ai déclaré une route par réseau, pointant vers l'adresse de transit du commutateur. C'est le chemin de retour : sans lui, les demandes partent mais les réponses ne reviennent pas.


**По-русски.** Обязательно скажи «chemin de retour» — обратный путь. Это показывает, что ты мыслишь симметрией.




### Pourquoi la passerelle par défaut n'est-elle déclarée que sur l'interface WAN ?


**Réponse.** Parce que la passerelle par défaut répond à la question « où envoyer ce que je ne connais pas », et cette réponse ne peut être qu'unique. La déclarer sur deux interfaces rendrait le choix du chemin imprévisible.


**По-русски.** Коротко и уверенно. Один выход наружу — один шлюз по умолчанию.




### Quelle est la différence entre le NAT sortant et la redirection de port ?


**Réponse.** Le NAT sortant remplace l'adresse source quand le trafic part vers l'extérieur : les adresses privées ne sont pas routables sur Internet, donc le pare-feu met la sienne à la place. La redirection de port fait l'inverse, à l'entrée : elle remplace l'adresse et le port de destination pour amener une demande venue de l'extérieur vers un serveur interne.


**По-русски.** Формулируй через направление: исходящий меняет источник, входящий меняет назначение.




### Pourquoi avoir choisi le NAT manuel plutôt qu'automatique ?


**Réponse.** Parce qu'en mode manuel la liste des réseaux qui sortent est explicite. En mode automatique, un nouveau réseau obtiendrait un accès à Internet du simple fait d'exister, sans décision de ma part. C'était aussi une exigence du sujet.


**По-русски.** Ответ «потому что так требовали» — половина ответа. Первая половина: явный контроль над тем, кто выходит наружу.




### Pourquoi la DMZ n'est-elle pas traduite ?


**Réponse.** Parce que le serveur web n'a aucun besoin d'aller sur Internet : il répond aux demandes entrantes. Ne pas le traduire ajoute une barrière si le serveur venait à être compromis.


**По-русски.** Это вопрос «а почему не все сети?». Ответ — по необходимости, а не по привычке.




### Le pare-feu lui-même a-t-il besoin de NAT ?


**Réponse.** Non. Il émet déjà ses paquets avec l'adresse de son interface WAN, qui est routable en amont. Le NAT ne concerne que les réseaux situés derrière lui.


**По-русски.** Каверзный вопрос. Многие на автомате добавляют правило для самого firewall — и не могут объяснить зачем.




### Pourquoi avoir désactivé le blocage des réseaux privés sur le WAN ?


**Réponse.** Parce que l'interface WAN est elle-même adressée dans une plage privée : c'est le réseau de la salle de formation. Si je laissais l'option active, le pare-feu bloquerait son propre réseau de sortie. En production, face à un vrai fournisseur d'accès, cette option resterait activée.


**По-русски.** Обязательно добавь последнюю фразу про продакшн. Без неё выглядит как ослабление защиты, с ней — как осознанное решение под условия стенда.





## Фильтрация



### Qu'est-ce qu'un pare-feu à état, et quelle conséquence pratique ?


**Réponse.** Il mémorise les connexions établies dans une table d'états. Conséquence pratique : quand j'autorise une demande vers l'extérieur, la réponse revient automatiquement. Il n'y a pas besoin d'écrire de règle de retour, et il ne faut surtout pas en écrire.


**По-русски.** Термин — *pare-feu à état*. Вывод про отсутствие обратных правил и есть то, что проверяют.




### Dans quel ordre les règles sont-elles évaluées ?


**Réponse.** De haut en bas, et c'est la première correspondance qui l'emporte : la suite n'est pas examinée. À la fin de la liste il y a un refus implicite — tout ce qui n'est pas explicitement autorisé est interdit.


**По-русски.** Два факта в одном ответе: первое совпадение и неявный запрет.




### Pourquoi la règle vers le site interne est-elle placée avant le blocage des réseaux privés ?


**Réponse.** Parce que le serveur web se trouve lui-même dans une plage privée. Si le blocage était placé avant, il couperait aussi l'accès au site interne. En inversant ces deux lignes, le site interne devient inaccessible alors qu'Internet continue de fonctionner.


**По-русски.** Лучший вопрос, чтобы показать понимание порядка. Отвечай через последствие: что именно сломается.




### Pourquoi les postes clients ne peuvent-ils plus faire de ping vers Internet ?


**Réponse.** Parce que le sujet demandait les règles les plus restrictives possibles. Les clients ont besoin du web, donc j'autorise les ports 80 et 443, et rien d'autre. L'ICMP n'est pas nécessaire à leur usage. L'administrateur, lui, conserve un accès complet, ICMP compris, comme le sujet l'exige.


**По-русски.** Это не недоработка, а осознанное решение. Обязательно противопоставь админу — тогда видно, что политика избирательная.




### Pourquoi n'y a-t-il aucune règle sur l'interface DMZ ?


**Réponse.** Parce que le serveur n'a besoin d'initier aucune connexion. Les demandes qui viennent des clients ou de l'extérieur sont déjà suivies par la table d'états, et les réponses passent sans règle. Une interface sans règle signifie donc : rien ne part d'ici.


**По-русски.** Могут поймать: «а как тогда сайт отвечает?». Ответ — через таблицу состояний.




### Qu'est-ce que la règle anti-blocage, et pourquoi l'avoir désactivée ?


**Réponse.** C'est une règle intégrée qui laisse n'importe quelle source du réseau interne joindre le pare-feu sur les ports d'administration. Elle est évaluée avant mes règles, donc tant qu'elle est active, mon interdiction faite aux clients n'a aucun effet. Je l'ai désactivée, mais seulement après avoir écrit et vérifié un accès explicite pour le VLAN d'administration.


**По-русски.** Сильный ответ: не просто «отключил», а «отключил после того, как обеспечил себе доступ». Это показывает методичность.





## Проверки и каверзные вопросы



### Comment prouvez-vous que le site public est accessible depuis la salle ?


**Réponse.** En y accédant depuis une machine du réseau de la salle, pas depuis l'intérieur. La capture montre l'adresse du WAN dans la barre du navigateur et la page servie. Tester depuis l'intérieur ne prouverait rien : la demande pourrait emprunter un autre chemin et contourner la redirection.


**По-русски.** Тут проверяют методику доказательства. Фраза «depuis l'intérieur ne prouverait rien» — самая ценная в ответе.




### Un client ne reçoit pas d'adresse IP. Comment procédez-vous ?


**Réponse.** Je remonte les couches. D'abord le port : est-il dans le bon VLAN, le lien est-il actif ? Ensuite le trunk : le VLAN est-il autorisé jusqu'à l'équipement ? Ensuite le service : la plage existe-t-elle, reste-t-il des adresses libres ? Enfin la table des baux, pour voir si une demande est arrivée.


**По-русски.** Проверяют метод, а не знание. Отвечай слоями снизу вверх и не называй сразу причину.




### Vous ajoutez une règle, mais le comportement ne change pas. Pourquoi ?


**Réponse.** Deux causes possibles. Soit une connexion établie avant la modification survit dans la table d'états, et il faut attendre son expiration ou la supprimer. Soit une règle placée plus haut correspond déjà au trafic et l'emporte. Les compteurs de correspondance permettent de trancher.


**По-русски.** Назвать компетентно обе причины плюс способ различить — сильный ответ.




### Le pare-feu ne répond pas au ping. Est-il en panne ?


**Réponse.** Pas nécessairement. Le ping teste uniquement l'ICMP, qui peut être filtré. Il faut tester le service attendu : si le port d'administration répond, l'équipement fonctionne. Conclure à une panne à partir d'un seul ping est une erreur classique.


**По-русски.** Классическая ловушка. Ответ показывает, что ты различаешь протокол и доступность узла.




### Pourquoi avoir masqué les empreintes de mots de passe dans le rapport ?


**Réponse.** Parce que ce sont des condensats cassables par force brute. Les publier dans un document remis annulerait une partie du travail de sécurisation décrit dans ce même document.


**По-русски.** Вопрос на зрелость. Ответ показывает, что ты думаешь о документе как о носителе риска.




### Qu'amélioreriez-vous avec plus de temps ?


**Réponse.** Trois choses. Passer les deux sites en HTTPS avec des certificats. Mettre en place une supervision et une centralisation des journaux, pour détecter un incident sans le chercher à la main. Et automatiser la sauvegarde des configurations, aujourd'hui faite manuellement.


**По-русски.** Никогда не отвечай «ничего». Три конкретных пункта показывают, что ты видишь границы своей работы. Не называй то, что требовалось и не сделано — только то, что сверх задания.





## Словарь для устного ответа


Слова, на которых чаще всего спотыкаются. Если термин не вспомнится, ответ рассыплется, даже когда суть понятна.


        | Français | Русский  |


          | la trame / le paquet | кадр / пакет  |

          | l'étiquette 802.1Q | тег 802.1Q  |

          | le domaine de diffusion | широковещательный домен  |

          | la passerelle par défaut | шлюз по умолчанию  |

          | la table de routage | таблица маршрутизации  |

          | le chemin de retour | обратный путь  |

          | le bail DHCP | аренда адреса  |

          | le relais DHCP | ретрансляция DHCP  |

          | les redirecteurs | серверы пересылки DNS  |

          | le pare-feu à état | межсетевой экран с учётом состояний  |

          | la table d'états | таблица состояний  |

          | le refus implicite | неявный запрет  |

          | la première correspondance | первое совпадение  |

          | le NAT sortant | исходящая трансляция адресов  |

          | la redirection de port | проброс порта  |

          | le condensat / l'empreinte | хэш  |

          | l'attaque par double étiquetage | атака с двойным тегированием  |






**Готовность к защите**

        - [ ] Отвечаю на все вопросы вслух, не подглядывая
        - [ ] Каждый ответ строю как «что сделал → почему → чем доказано»
        - [ ] Знаю все термины из словаря и произношу без запинки
        - [ ] Могу показать в отчёте страницу под любое своё утверждение
        - [ ] Готов ответ на «что бы улучшили» — три конкретных пункта