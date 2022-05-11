# Predikcia trajektórie vozidiel pre asistenčné systémy riadenia 

Tento projekt sa zameriava na predikciu trajektórie vozidiel za použitia monokulárneho obrazu datasetu KITTI.

## Motivácia
Táto práca sa okrem samotnej predikcie pohybu vozidiel venuje aj problematike
tvorby vhodnej reprezentácie okolitého prostredia. Jej cieľom je konať len za použitia monokulárneho záznamu, ktorý oproti použitiu drahých senzorov vyžaduje len jednu kameru.
Na základe toho navrhujem model pozostávajúci z dvoch častí. Prvá časť
je zodpovedná za tvorbu vhodnej reprezentácie prostredia a druhá časť spracúva túto reprezentáciu a vytvára predikciu.




## Funkcie



## Inštalácia

Systém bol vyvíjaný na Ubuntu 18.04 64-bit pomocou Python 3.8.

Pre vytvorenia prostredia conda slúži príkaz 

`conda create --name <env_name> --file requirements.txt --python=3.8`

Prostredie sa aktivuje príkazom

`conda activate <env_name>`

1. Nainštalovať [ORB_SLAM2](https://github.com/raulmur/ORB_SLAM2)
2. Na fungovanie ORB_SLAM2 v prostredí Python je potrebné nainštalovať [ORB_SLAM2-PythonBindings](https://github.com/jskinn/ORB_SLAM2-PythonBindings)

Pre beh systému je potrebná grafická karta s výpočtovou platformou CUDA.

### Modely pre predspracovanie

Používané vytrénované modely sú dostupné na odkaze https://drive.google.com/drive/folders/1yCVw2ORS1v4qe3tx9Lv5e1eYRI6nP1oQ?usp=sharing.

Tieto modely je potrebné umiestniť podľa nasledujúceho rozdelenia:

mono+stereo_640x192: `depth_estimation/monodepth2/models/`

panoptic_fpn_R_101_3x.pkl: `semantic_segmentation/`

test.pth: `object_detection_3d/GUPNet/code/checkpoints/`

### Predikčné modely

Vytrénované predikčné modely sú dostupné na odkaze https://drive.google.com/file/d/1FrGpkrq2iCrAwgR5_tMNF7jD7llFWl2E/view?usp=sharing. Obsah `.zip` súboru je potrebné extrahovať do zložky 
`prediction/` 

## Príprava datasetu

Systém pracuje s monokulárnym záznamom datasetu KITTI, konkrétne so scénami z časti kiti_raw a kitti_tracking. Pred použitim programu je potrebné zorganizovať súborovú štruktúru datasetu podľa nasledujúcej formy.

```
├── kitti_raw
|   ├── 2011_09_26
|   |   |   ...
|   │   └── 2011_09_26_drive_0009_sync
|   │       └── image_02
|   │           └── data
|   ├── 2011_09_28
|   |   |   ...
|   │   └── 2011_09_28_drive_0002_sync
|   │       └── image_02
|   │           └── data
|   ├── 2011_09_29
|   |   |   ...
|   │   └── 2011_09_29_drive_0071_sync
|   │       └── image_02
|   │           └── data
|   ├── 2011_09_30
|   |   |   ...
|   │   └── 2011_09_30_drive_0034_sync
|   │       └── image_02
|   │           └── data
|   └── 2011_10_03
|       |   ...
|       └── 2011_10_03_drive_0047_sync
|           └── image_02
|               └── data
|
└── kitti_tracking
    └── data_tracking_image_2
        ├── testing
        │   └── image_02
        │       |   ...
        │       └── 0017
        └── training
            └── image_02
                |   ...
                └── 0015
```

Dáta spracované do formy používanej predikčným modelom sú dostupné na odkaze https://drive.google.com/drive/folders/1Nj-Wa8nCbfe2yfTS3imTGUKzEZJ7v7Yw?usp=sharing.

## Predspracovanie obrazu

V konfiguračných `config/*.yaml` súboroch treba pred spustením skriptov na spracovanie obrazu nastaviť adresár s datasetom KITTI a cieľový adresár, kam sa majú spracované dáta uložiť.

Obraz s pôvodnou veľkosťou sa spracuje pomocou príkazu

`python generate_dataset.py config/kitti_big.yaml`

Zmenšený obraz sa spracuje príkazom

`python generate_dataset.py config/kitti_small.yaml`

## Trénovanie predikčných modelov

Konfigurácie trénovaných predikčných modelov sa nachádzajú v adresári `prediction/config`. `.yaml` súbory určené pre trénovanie na dátach získaných z obrazu pôvodnej veľkosti sa nachádzajú v zložke `big_input`. Pre konfiguráciu trénovania modelov zo zmenšených obrázkov je možné použiť `.yaml` súbory zo zložky `small_input`. V jednotlivých konfiguračných súboroch treba pred samotným trénovaním nastaviť zložku s predspracovanými dátami. 



Pred samotným spustením skriptov je potrebné sa presunúť do zložky `prediction/`. Pre každú konfiguráciu treba vytrénovať časti modelu v tomto poradí

1. auto-encoder: `python train_autoencoder.py <config>`
2. memory-controller: `python train_memory_controller.py <config>`
3. iterative refinement module: `python train_itrefmodule.py <config>`

Pomocou skriptu `train_all.sh` je možné vytrénovať modely pre každú konfiguráciu.

## Testovanie modelu
Tento systém je oproti špičkovým metódam nedostatočný a dopúšťa sa chyby, ktorá je v priemysle neakceptovateľná. Taktiež, čas spracovania obrazu nedosahuje spoľahlivosť, ktorá by sa približovala výkonu v reálnom čase. 

Pred samotným spustením skriptov pre testovanie je potrebné sa presunúť do zložky `prediction/`. 
Modely je možné otestovať pomocou príkazu

`python test_configs.py <config>`

Pomocou skriptu `test_all.sh` je možné otestovať modely vytvorené každou z konfiguráciií.


## Vizualizácia


Vizualizáciu premietnutých predikcií do obrazu je možné spustiť pomocou skriptu

`python run.py <prediction_config> <preprocessing_config> <data_path>`

`<preprocessing_config>` je cesta ku niektorému z konfiguračných súborov v zložke `config`.

`<prediction_config>` je cesta ku niektorému z konfiguračných súborov v zložke `prediction/config`.

`<data_path>` cesta k zložke so súbormi predspracovanej scény

### Predikcia 1s → 2s

![](doc/scene_10_20.gif)

### Predikcia 2s → 1s
![](doc/scene_20_10.gif)



## Referencie

Odhad hlbky - [monodepth2](https://github.com/nianticlabs/monodepth2)

Sémantická segmentácia - [detectron2](https://github.com/facebookresearch/detectron2)

3D detekcia vozidiel - [GUPNet](https://github.com/SuperMHP/GUPNet)

Lokalizácia kamery - [ORB_SLAM2](https://github.com/raulmur/ORB_SLAM2)

Python bindings pre SLAM -[ORB_SLAM2-PythonBindings](https://github.com/jskinn/ORB_SLAM2-PythonBindings)

Predikcia - [MANTRA-CVPR20](https://github.com/Marchetz/MANTRA-CVPR20)
