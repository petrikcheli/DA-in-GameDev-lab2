# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил(а):
- Артемьев Петр Александрович
- РИ210950
Отметка о выполнении заданий (заполняется студентом):

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

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Познакомиться с программными средствами для организции передачи данных между инструментами google, Python и Unity

## Задание 1
### Реализовать совместную работу и передачу данных в связке Python - Google-Sheets – Unity. 
#### 1) В облачном сервисе google console подключить API для работы с google sheets и google drive.
- В облачном сервисе "google console" авторизовался и создал новый проект.
![1](https://user-images.githubusercontent.com/126482589/230716721-a5cc61c7-8248-45ee-b348-075244fbc74f.png)

![2](https://user-images.githubusercontent.com/126482589/230716724-65a1b87f-5046-460c-a712-5d47acf6ca8e.png)


- Подключил API для работы с google sheets и google drive.
![3](https://user-images.githubusercontent.com/126482589/230716740-54f41ac2-bda2-44ef-897f-fb1bd31e2364.png)


- Создал сервисный аккаунт и ключ.
![4](https://user-images.githubusercontent.com/126482589/230716772-f9f48e97-b9f2-480b-98b6-123113c3818f.png)


- Создал документ в google таблицах и предоставил доступ сервисному аккаунту.
![5](https://user-images.githubusercontent.com/126482589/230716788-ace96bea-56f7-40f5-be6e-0e27958c0338.png)


#### 2) Реализовать запись данных из скрипта на python в google-таблицу. Данные описывают изменение темпа инфляции на протяжении 11 отсчётных периодов, с учётом стоимости игрового объекта в каждый период.
- После запуска кода - полученные данные появились в таблице
![6](https://user-images.githubusercontent.com/126482589/230716804-07541a63-c13e-42e7-a50e-066b7cd268ac.png)
![7](https://user-images.githubusercontent.com/126482589/230716811-e2b109dd-c437-4662-821c-bd8223c5c9fd.png)


#### 3) Создать новый проект на Unity, который будет получать данные из google-таблицы, в которую были записаны данные в предыдущем пункте.
- Создал новый проект и импортировал пакеты из материалов к лабораторной работе.
![8](https://user-images.githubusercontent.com/126482589/230716824-5e13a397-a6c2-4f09-8afb-557c31b0be16.png)


#### 4) Написать функционал на Unity, в котором будет воспризводиться аудио-файл в зависимости от значения данных из таблицы.
- Прописал в скрипте код из методических материалов к ЛР.
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] <= 10 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1KFC3nba9FJfv6Vq3GHt2nLatDDzqGkECBC2vi36s-_M/values/Лист1?key=AIzaSyBUMKnWhpCaNxjFiY8T3mJ2_3Ds5lJmEbE");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}
```

- Добавил музыку из импортированных файлов к коду
![9](https://user-images.githubusercontent.com/126482589/230716854-fd5b6206-b9be-40bf-8bfa-d8ad47ecaa94.png)


- Вывод программы:
![10](https://user-images.githubusercontent.com/126482589/230716867-9ab63ce6-2742-42f3-998a-e2b747ce689f.png)
С каждой строкой вывода воспроизводится звук, соответствующий значению строки гугл-таблицы.


## Задание 2
### Реализовать запись в Google-таблицу набора данных, полученных с помощью линейной регрессии из лабораторной работы № 1. 
- Поправил код из ЛР-1

```py
import gspread
import numpy as np
import matplotlib.pyplot as plt

x = [3, 21, 22, 34, 54, 34, 55, 67, 89, 99]
x = np.array(x)
y = [2, 22, 24, 65, 79, 82, 55, 130, 150, 199]
y = np.array(y)

plt.scatter(x, y)


def model(a, b, x):
    return a * x + b


def loss_function(a, b, x, y):
    num = len(x)
    prediction = model(a, b, x)
    return (0.5 / num) * (np.square(prediction - y)).sum()


def optimize(a, b, x, y):
    num = len(x)
    prediction = model(a, b, x)
    da = (1.0 / num) * ((prediction - y) * x).sum()
    db = (1.0 / num) * ((prediction - y).sum())
    a = a - Lr * da
    b = b - Lr * db
    return a, b


def iterate(a, b, x, y, times):
    for i in range(times):
        a, b = optimize(a, b, x, y)
    return a, b

a = np.random.rand(1)
b = np.random.rand(1)
Lr = 0.000001

gc = gspread.service_account(filename='unitydatascience-365118-7456c414ecb4.json')
sh = gc.open("UnitySheets")
price = np.random.randint(2000, 10000, 11)
mon = list(range(1,11))
i = 0
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        a, b = iterate(a, b, x, y, 100)
        prediction = model(a, b, x)
        loss = loss_function(a, b, x, y)
        loss = str(loss).replace('.', ',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(price[i-1]))
        sh.sheet1.update(('C' + str(i)), str(loss))
        print(loss)
```

- Добавил ссылку-ключ на новый проект
![11](https://user-images.githubusercontent.com/126482589/230716906-66f87363-4ca6-4358-806c-b690ede73ad3.png)


- Запустил код, чтобы записать в таблицу новые данные.
![12](https://user-images.githubusercontent.com/126482589/230716919-42447439-5912-4733-a6f8-6b329d71b1d6.png)
![13](https://user-images.githubusercontent.com/126482589/230716931-5ecbe9f4-a051-4837-9de9-879e07b3532c.png)


## Задание 3
### Самостоятельно разработать сценарий воспроизведения звукового сопровождения в Unity в зависимости от изменения считанных данных в задании 2.
- Изменил пограничные значения, при которых воспроизводятся файлы.
![14](https://user-images.githubusercontent.com/126482589/230716945-8db3e11a-3461-4003-bfc7-2d4b80bef9b4.png)


- Вывод в Unity
![15](https://user-images.githubusercontent.com/126482589/230716966-f0952100-9247-40c6-a671-3d5a70697c08.png)
Так же воспроизводится соответствующая аудиодорожка в зависимости от значения в таблице.


| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
