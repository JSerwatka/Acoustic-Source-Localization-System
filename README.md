## Spis treści
- [Cele projektu](#cele-projektu)
- [Opis działania programu](#opis-działania-programu)
- [Metoda pomiaru TDoA](#metoda-pomiaru-tdoa)
- [Wyliczanie kąta](#wyliczanie-kąta)
- [Panel użytkownika](#panel-użytkownika)


### Cele projektu
Celem poniższej pracy jest zaprojektowanie i wykonanie systemu lokalizacji źródła akustycznego (ang. _acoustic source localization system_) z wykorzystaniem oprogramowania _LabVIEW _oraz urządzenia _sbRIO-9636_ od _National Instruments_. Przyrząd ma za zadanie podanie lokalizacji dźwięku (ang.
_Direction of Arrival_ (DoA)) w czasie rzeczywistym, w formie odchylenia kąta względem środka urządzenia. Cały system składa się z trzech głównych elementów: czterech mikrofonów, czterech wzmacniaczy mikrofonowych oraz _sbRIO-9636_. Wspomniany wyżej kąt, jest wyliczany na podstawie opóźnień sygnałów akustycznych (ang. _Time Difference of Arrival_(TDoA)), pomiędzy kolejnymi parami mikrofonów. Dalej, z zebranych w ten sposób danych, można wyliczyć położenie źródła dźwięku względem środka urządzenia.

Dodatkowym celem pracy jest wykonanie opisywanego tu projektu przy wykorzystaniu możliwie najtańszych komponentów, ale jednocześnie oferującego zadowalająca˛ dokładność wyliczeń w trudnych warunkach akustycznych. Korzystanie z niskiej jakości elementów tworzy wiele komplikacji, ale umożliwia łatwa powielalność projektu.

### Opis działania programu
Poniżej znajduje się uproszczony opis kolejnych kroków działania systemu:
1. Po wzmocnieniu sygnału, jest on próbkowany z wybraną częstotliwością (domyślnie – 50 kS/s).
2. Sygnał jest filtrowany filtrem górnoprzepustowym o częstotliwości granicznej 2:5 kHz.
3. Gdy dźwięk przekroczy określony próg, gromadzona jest wybrana przez użytkownika liczba próbek
(domyślnie – 9000).
4. Zebrany sygnał dźwięku jest kolejkowany w FIFO.
5. Pętla deterministyczna, pracująca w części programu, która operuje w czasie rzeczywistym, odbiera
dane z FIFO i przekazuje je do wewnętrznej kolejki (RT FIFO).
6. Dane odbierane są w niedeterministycznej pętli, zajmującej się analizą sygnału.
7. Przy użyciu _Generalized Cross Correlation_ z _Phase Transform Weights_ mierzony jest _TDoA_.
8. Na podstawie danych z poprzedniego punktu, wyliczany jest kąt padania dźwięku na każdą parę
mikrofonów.
9. Kąty są analizowane w celu sprawdzenia, czy nie wystąpił błąd pomiaru.
10. Wyliczany jest ostateczny kąt względem środka matrycy.
11. Dane o kącie przesyłane są do panelu użytkownika.
12. Na panelu użytkownika wyświetlane są następujące dane:
    * ostateczny kąt padania dźwięku,
    * wykres badanego sygnału dla jednego z mikrofonów,
    * wykres TDoA dla jednej z par mikrofonów.

![diagram](https://user-images.githubusercontent.com/33938646/71518613-0a48f800-28b4-11ea-8894-7f0e0f73baa9.png)


### Metoda pomiaru TDoA
Aby wytłumaczyć działanie GCC należy najpierw stworzyć model matematyczny sygnałów odbieranych przez mikrofony:
![Screenshot_3](https://user-images.githubusercontent.com/33938646/71518762-a70b9580-28b4-11ea-8487-752a68647630.png)

Podstawową metodą wyliczania TDoA jest _Cross-correlation_ (CC). Polega ona na przeprowadzeniu funkcji korelacji wzajemnej (ang. _cross-correlation_) na dwóch odebranych sygnałach. Dzięki temu jesteśmy w stanie otrzymać wykres podobieństwa dwóch sygnałów dla różnych przesunięć czasowych. Dalej, należy tylko znaleźć wartość dla której funkcja jest w swoim maksimum, i otrzymujemy szukaną wielkość opóźnienia między sygnałami. Metoda CC jest niezwykle prosta, ale jednocześnie bardzo zawodna, szczególnie przy niskiej wartości SNR.

W celu zwiększenia dokładności wyników należy skorzystać z odpowiedniej filtracji i funkcji nadającej wagę (ang. _weighting function_), aby wzmocnić istotne elementy sygnału i wytłumić niepotrzebny szum. W ten sposób otrzymujemy _Generalized Cross-Correlation_, którą można przedstawić za pomocą następującego równania:
![Screenshot_4](https://user-images.githubusercontent.com/33938646/71518874-4b8dd780-28b5-11ea-8fdf-c8c05ccb2394.png)

Najczęściej badanymi weighting functions są: _Roth_, _Smoothed COherence Transform_ (SCOT), _Phase_
_Transform_ (PHAT) oraz _Eckart Filter_. Wiele z tych analiz wyraźnie wskazuje, że najbardziej skuteczną
metodą jest GCC-PHAT. To podejście jest też najczęściej używane w podobnych projektach. Charakteryzuje się ono prostotą obliczeń, przy jednoczesnej dużej dokładności w szacowaniu TDoA, przez co jest niezwykle użyteczna w systemach lokalizacji dźwięku działających w czasie rzeczywistym. PHAT można przedstawić za pomocą następującego równania:
![Screenshot_22](https://user-images.githubusercontent.com/33938646/71519381-9f99bb80-28b7-11ea-898d-7dc9f9d16523.png)


Schemat blokowy metody oszacowania TDoA, z wykorzystaniem _generalized_
_cross-correlatio_n:
![GCC](https://user-images.githubusercontent.com/33938646/71519207-d28f7f80-28b6-11ea-8b23-c52e10f75539.jpg)

### Wyliczanie kąta
Znajomość TDoA pozwala na obliczenie kąta padania fali akustycznej na parę mikrofonów. Wykorzystuje
się w tym celu proste działanie trygonometryczne.
![Screenshot_6](https://user-images.githubusercontent.com/33938646/71519258-1e422900-28b7-11ea-884a-708205e9599d.png)
![Screenshot_7](https://user-images.githubusercontent.com/33938646/71519275-344fe980-28b7-11ea-9f2b-1b2ef4714fd6.png)
![Screenshot_8](https://user-images.githubusercontent.com/33938646/71519310-58abc600-28b7-11ea-860f-2800003a3c87.png)

### Panel użytkownika
![Screenshot_3](https://user-images.githubusercontent.com/33938646/71519497-0919ca00-28b8-11ea-9031-90a42b20275a.png)
![Screenshot_4](https://user-images.githubusercontent.com/33938646/71519502-0f0fab00-28b8-11ea-9c01-2bbff95a6d90.png)

![panel](https://user-images.githubusercontent.com/33938646/71519454-dd96df80-28b7-11ea-8ebd-bd3964e51073.jpg)