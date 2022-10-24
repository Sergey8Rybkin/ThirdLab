# Лабораторная работа № 3 Реализация интерфейса пользователя

Отчет по лабораторной работе #3 выполнил:
- Рыбкин Сергей Денисович
- РИ-300012

[Репозиторий с проектом](https://github.com/Sergey8Rybkin/Dragon-Picker)

[Черновик игры на Яндекс Играх](https://yandex.ru/games/app/198497?draft=true&lang=ru)

Отметка о выполнении заданий:

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

Цель работы: интеграция интерфейса пользователя в разрабатываемое интерактивное приложение.

### Задание 1.
Используя видео-материалы практических работ 1-5 повторить реализацию игровых механик: 
–1 Практическая работа «Реализация механизма ловли объектов».
–2 Практическая работа «Реализация графического интерфейса с добавлением счетчика очков».

### Задание 2.
–3 Практическая работа «Уменьшение жизни. Добавление текстур».
–4 Практическая работа «Структурирование исходных файлов в папке».


Мы продолжаем работать в уже существующем репозитории и улучшаем наш проект.

Закончили мы на создании скрипта спавнящего наши щиты, что так-же будет жизнями нашего игрока. Поэтому начнём с создания управления нашей каретки. 
Управлять будем мышкой. В дальнейшем это поможет нашему игроку быстро реагировать на стремительно падающие яйца. Пишем код в апдейт, тогда обрабатываеться будет управление каждый кадр.

```c#
void Update()
    {
        Vector3 mousePos2D = Input.mousePosition;
        mousePos2D.z = -Camera.main.transform.position.z;
        Vector3 mousePos3D = Camera.main.ScreenToWorldPoint(mousePos2D);
        Vector3 pos = this.transform.position;
        pos.x = mousePos3D.x;
        this.transform.position = pos;
        }
```
## Результат

![unknown_2022 10 24-22 12_1](https://user-images.githubusercontent.com/100475554/197585955-8158b423-ed71-4a4a-98f8-f6e46c6dd366.gif)

Дальше займёмся тем, чтобы яйца приземлившись не катались на шаре, а поглощались. 
При колизии с щитом наше яйцо должно пропадать

```c#
 private void OnCollisionEnter(Collision coll) {
            GameObject Collided = coll.gameObject;
            if (Collided.tag == "DragonEggs") {
                Destroy(Collided);
            }
            }
```
В итоге получаем ловлю объектов. Осталось накрутить очки

![unknown_2022 10 24-22 19_1](https://user-images.githubusercontent.com/100475554/197587359-3a1e45c9-65fd-4c54-9772-af22e106789b.gif)

Сначала для подсчёта очков стоит создать первые элементы UserInterface. Для них мы можем использовать **Canvas**.
Создаём его вместе с текстом. Настраиваем расположение и настройки текста.

![image](https://user-images.githubusercontent.com/100475554/197588873-b34a4c11-f5a1-4fd0-b536-e5b6eaa054b4.png)

![image](https://user-images.githubusercontent.com/100475554/197588758-d35c9243-3047-4ecd-b5f1-4466c2fcf120.png) ![image](https://user-images.githubusercontent.com/100475554/197589110-b6de6246-80ab-4684-8dab-8fec71d335ae.png)

Теперь когда мы подключили UI на который мы будем выводить результат пользователя, осталось только реализовать подсчёт его очков, по одному за каждое яйцо. Из важных элементов тут это подключение **using TMPro;** для работы с графическими элементами Unity.
Далее прописываем новую переменную подсчёта и сам скрипт подсчёта очков при каждом колайде яиц со щитом. Конечная версия скрипта щита будет выглядеть так

```c#
using UnityEngine;
using TMPro;
public class EnergyShield : MonoBehaviour
{

    public TextMeshProUGUI scoreGT;
    // Start is called before the first frame update
    void Start()
    {
        GameObject scoreGo = GameObject.Find("Score");
        scoreGT = scoreGo.GetComponent<TextMeshProUGUI>();
        scoreGT.text = "0";
    }

    
    void Update()
    {
        Vector3 mousePos2D = Input.mousePosition;
        mousePos2D.z = -Camera.main.transform.position.z;
        Vector3 mousePos3D = Camera.main.ScreenToWorldPoint(mousePos2D);
        Vector3 pos = this.transform.position;
        pos.x = mousePos3D.x;
        this.transform.position = pos;
        }

       
        private void OnCollisionEnter(Collision coll) {
            GameObject Collided = coll.gameObject;
            if (Collided.tag == "DragonEggs") {
                Destroy(Collided);
            }
            int score = int.Parse(scoreGT.text);
            score += 1;
            scoreGT.text = score.ToString();
            }
            
}
```
Ура, теперь наш игрок может видеть результат своей игры.

![unknown_2022 10 24-22 38_1](https://user-images.githubusercontent.com/100475554/197590766-1c15af4b-49d3-4f58-80e8-ea73b95bc2a1.gif)

Но игра где нет конца, и нет возможности проиграть бессмыслена. Поэтому нам нужно ввести "наказание" за то что игрок пропускает яйцо. Через три пропуска игра должна заканчиваться, и игрок начинает сначала.

Для этого нам необходимо :
- получить информацию о том что яйцо уничтожено
- в главном скрипте игры сделать реакцию на уничтожение яйца
- уменьшить количество щитов игрока (количество жизней)
- когда количество щитов (жизней) упадёт до нуля перезапускаем уровень

Вносим изменения в скрипт яйца, создаём метод который сможем вызывать в главном скрипте игры

```c#
void Update()
    {
        if (transform.position.y < bottomY) {
            Destroy(this.gameObject);
            DragonPicker apScript = Camera.main.GetComponent<DragonPicker>();
            apScript.DragonEggDestroyed();
        }
    }
```

В главном скрипте **Dragon Picker** мы уже создали метод создающий щиты. Теперь нам нужно их посчитать, и за каждое падение уничтожать один из них. Тут нам понадобится новый инструмент позволяющий воздействовать на сцену. Так будет выглядеть наш скрипт теперь.

```c#
using UnityEngine;
using UnityEngine.SceneManagement;

public class DragonPicker : MonoBehaviour
{
    public GameObject energyShieldPrefab;

    public int numEnergyShield = 3;

    public float energyShieldBottomY = -6f;

    public float energyShieldRadius = 1.5f;

    public List<GameObject> shieldList;
    // Start is called before the first frame update
    void Start()
    {
        for (int i = 1; i <= numEnergyShield; i++){
            GameObject tShieldGo = Instantiate<GameObject>(energyShieldPrefab);
            tShieldGo.transform.position = new Vector3(0, energyShieldBottomY, 0);
            tShieldGo.transform.localScale = new Vector3(1*i, 1*i, 1*i);
            shieldList.Add(tShieldGo);
        }
    }

    
    void Update()
    {
        
    }
    public void DragonEggDestroyed() {
        GameObject[] tDragonEggArray = GameObject.FindGameObjectsWithTag("DragonEggs");
        foreach (GameObject tGO in tDragonEggArray) {
            Destroy (tGO);
        }
        int shieldIndex = shieldList.Count -1 ;
        GameObject tShieldGo = shieldList [shieldIndex];
        shieldList.RemoveAt(shieldIndex);
        Destroy(tShieldGo);

        if (shieldList.Count == 0) {
            SceneManager.LoadScene("_0Scene");
        }
    }
    
}
```

## Результат

![unknown_2022 10 25-00 04_1](https://user-images.githubusercontent.com/100475554/197605822-e49be978-706f-475a-a257-0109eedf741f.gif)

Теперь, когда наш геймплей выглядит ещё привлекательнее, пора улучшить и вшенюю составлющую нашей игры.
Нас интересует новый ассет пак ***Autumn Mountain***
1. Загружаем и импортируем
2. Добавляем префаб горы и располагаем её на сцене
3. Добавляем скайбокс в разделе **Lighting**

![image](https://user-images.githubusercontent.com/100475554/197606508-63d683dd-4c3a-4547-8640-b6bbdc11e640.png)

![image](https://user-images.githubusercontent.com/100475554/197606968-2c0ae5c2-6c06-4173-b9a3-d01ee425a3f2.png)

Теперь наша игра выглядит ещё живее, после замены дефолтного скайбокса юнити на горную местность. 

### Дракон очень рад
![unknown_2022 10 25-00 15_1](https://user-images.githubusercontent.com/100475554/197608092-a1aac669-46d7-4811-b251-410c1301d521.gif)

Теперь перед выкладыванием нашего проета на Яндекс Игры нам стоит прибраться в репозитории. Любой лишний мегабайт может повлиять на опыт пользователя.
Для начала создаём папки для каждого объекта нашей игры
+ Animations
+ AudioFiles
+ Materials
+ Prefabs
+ Scenes
+ Scripts
+ Textures

Из нашей папки файлы переносим в надлежащие папочки. Так-же нам нужно из импортированных ассетов выбрать все необходимые файлы и так-же перенести в соответсвующие папки.
1. Текстуры, материалы и анимации дракона
2. Материалы, текстуры и карты для них из *Fire and Spell Effects*
3. Префаб, материаллы и папку скайбокса из папки с горой.

Когда все необходимые файлы перенесены можно удалить папки с ассетами.

### Получаем красивый чистый раздел с контентом нашей игры
![image](https://user-images.githubusercontent.com/100475554/197609995-37f83407-bc24-42ec-9c11-eac1e9756271.png)

Все изменения добавлены в [Репозиторий с проектом](https://github.com/Sergey8Rybkin/Dragon-Picker)

### Задание 3.
–	5 Практическая работа «Интеграция игровых сервисов в готовое приложение».

Мы делаем нашу игру под WebGL, поэтому первым делом скачиваем модуль для работы в этом направлении. По загрузке пробуем создать свой первый билд
1. Добавляем нашу сцену
2. Настраиваем графику
3. Настраиваем компрессию

![image](https://user-images.githubusercontent.com/100475554/197612421-121d141d-5d38-4689-ac7f-ee0b69640220.png) ![image](https://user-images.githubusercontent.com/100475554/197612482-63da9dcb-f152-4c24-96fd-62023c67e5ca.png)


