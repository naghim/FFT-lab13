# Labor 13

## Threadek

Qt-ban lehetőségünk adódik a processzorunkat teljes mértékben kihasználni párhuzamos szálak segítségével. Qt-ban az erre használható osztály a <a href="https://doc.qt.io/qt-6/qthread.html">QThread</a> osztály. A <a href="https://doc.qt.io/qt-6/qthread.html">QThread</a> egy absztrakt osztály, amit származtatnunk kell, és implementálnunk kell a <a href="https://doc.qt.io/qt-6/qthread.html#run">QThread::run()</a> függvényét. Ez lesz az a függvény, ami végrehajtásra kerül a párhuzamos szál elinditása után. Egy párhuzamos szálat a <a href="https://doc.qt.io/qt-6/qthread.html#start">QThread::start()</a> függvénnyel indíthatunk el. Vigyázzunk arra, hogy a threadet ne a `run` függvénnyel, hanem a `start` függvénnyel indítsuk el. Ha a `run` függvénnyel indítjuk, ugyanazon a szálo fog futni, mint a felhasználói felület (GUI thread).

A <a href="https://doc.qt.io/qt-6/qthread.html">QThread</a> akkor is hasznos, hogyha a háttérben szeretnénk valamit futtatni. Ha egy kódrészletet egy <a href="https://doc.qt.io/qt-6/qthread.html">QThread</a>-ben indítunk el, az egy teljesen független szálon fog futni a felhasználói felülettől. Ha egy hosszabb kódot elindítunk simán a Qt-ből, akkor le fogja fagyasztani a felületet. Ha <a href="https://doc.qt.io/qt-6/qthread.html">QThread</a>-ből indítjuk, akkor, mivel a háttérben fut egy teljesen más szálon, nem fogja befolyásolni a felület futtatását, vagyis responsive marad a felület.

Egy <a href="https://doc.qt.io/qt-6/qthread.html">QThread</a>-en belül sajnos nem tudjuk módosítani a felület kinézetét. Például nem tudunk hozzáférni a felület komponenseihez. Ha például egy progress bárt szeretnénk frissíteni, azt közvetlenül a threadből nem tudjuk. Emiatt szükséges a signal-slot mechanizmus használata. A <a href="https://doc.qt.io/qt-6/qthread.html">QThread</a> képes küldeni egy signal-t, amit kifoghatunk a felhasználói felület részéről egy megfelelő slottal. Például:

```cpp
class WorkerThread : public QThread {
    Q_OBJECT

public:
    void run() override;

signals:
    void progressUpdated(int value);
};

void WorkerThread::run() {
    // Hosszabb időt felvevő kód futtatása...
    // ...

    // Frissítsük a felületet signal segítségével
    emit progressUpdated(50);
}
```

illetve:

```cpp
class MainWindow : public QMainWindow {
    Q_OBJECT

private slots:
    void onProgressUpdated(int value);

private:
    void startThread();
};

void MainWindow::onProgressUpdated(int value) {
    // Az esemény feldolgozása
    this->progressBar->setValue(progress);
}

void MainWindow::startThread() {
    WorkerThread *thread = new WorkerThread;

    // Feliratkozás az eseményre
    connect(thread, &WorkerThread::progressUpdated, this, &MainWindow::onProgressUpdated);

    // A szál konkrét elindítása
    thread->start();
}
```

## Feladatok

<p>
  1. Készítsünk egy programot amely segítségével a felhasználó egy fájlban található számok közül meg tudja keresni a leghosszabb Collatz-pályát. </p>

A **Collatz-sejtés** a következőket mondja ki: vegyünk egy tetszőleges pozitív egész számot és a következő műveleteket egymás után hajtsuk végre rajta, a végén mindig 1-hez érünk:

- ha a szám páros, osszuk el kettővel;

- ha a szám páratlan, háromszorozzuk meg, és adjunk hozzá egyet.

Vagyis a következő képlettel írhatjuk ezt le:

<p style="align: center">
<img src="https://i.imgur.com/dKe4a3Y.png" width="300px"></p>

**Collatz-pályának** nevezzük azt a számszekvenciát, amelyet az adott számtól kiindulva, elvégezve a műveleteket kapunk egészen az 1-ig bezárólag. A lépések/számok darabszáma a pálya hossza.

_Például_:

`7` esetén a pálya a következőképp alakul:
`7, 22, 11, 34, 17, 52, 26, 13, 40, 20, 10, 5, 16, 8, 4, 2, 1`.

A szekvencia hossza pedig: `16`.

A felület adjon lehetőséget a felhasználónak kiválasztani egy állományt (használjunk <a href="https://doc.qt.io/qt-6/qfiledialog.html">QFileDialog</a>-ot), amely pozitív egész számokat tartalmaz (lásd `numbers.txt` a labor fájljai között). A Process gombra kattintva kapcsoljuk ki a felület gombjait, és indítsunk `N` darab (`N` a CPU magok száma) párhuzamos szálat (lásd <a href="https://doc.qt.io/qt-6/qthread.html">QThread</a>). A szálak között osszuk fel a beolvasott számokat, hogy mindegyik szál ugyanannyi számot dolgozzon fel:

```cpp
int numThreads = std::thread::hardware_concurrency();

std::vector<WorkerThread *> threads;
int chunkSize = numbers.size() / numThreads;
for (int i = 0; i < numThreads; ++i) {
    int start = i * chunkSize;
    int end = (i == numThreads - 1) ? numbers.size() : start + chunkSize;
    std::vector<int> chunk(numbers.begin() + start, numbers.begin() + end);
    WorkerThread *thread = new WorkerThread(chunk);
    connect(thread, &WorkerThread::progressUpdated, this, &MainWindow::onProgressUpdated);
    connect(thread, &WorkerThread::newLongestPathFound, this, &MainWindow::onNewLongestPathFound);
    threads.push_back(thread);
}
```

Mindegyik szál felelős a neki kiosztott számok Collatz-pályájának kiszámításáért. Minden 100 szám kipróbálása után emitteljen egy `progressUpdated` nevű signált, amelyet felfog az ablakunk és frissíti a rajta lévő progress bárt. Amennyiben kapunk egy új leghosszabb Collatz-pályát értesítsük az ablakot a `newLongestPathFound` signállal. Ha az összes számot kipróbáltuk, akkor oldjuk fel a felületen található gombokat, hogy lehessen újrapróbálni a számítást. Vigyázzunk arra, hogy az utolsó száz vagy kevesebb számot is lejelentsük miután feldolgoztuk őket.

<p align=center>
<img src="https://i.imgur.com/VEi1ju8.png" width="300px"></p>
