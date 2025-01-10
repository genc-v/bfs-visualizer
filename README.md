# Vizualizimi i BFS dhe DFS

Ky projekt i zhvilluar në **Nuxt.js** ofron një mjet interaktiv për të vizualizuar algoritmet e Kërkimit në Gjerësi (BFS) dhe Kërkimit në Thellësi (DFS). Projekti është i hostuar në Netlify dhe mund të shihet live në adresën: [ubt-matematike.netlify.app](https://ubt-matematike.netlify.app/).

## Përmbledhje

### Çfarë janë BFS dhe DFS?

#### Kërkimi në Gjerësi (BFS)
BFS (Breadth-First Search) është një algoritëm i përdorur për të eksploruar grafët apo pemët duke filluar nga një nyjë burimore dhe duke eksploruar çdo nyjë fqinje në të njëjtin nivel përpara se të kalojë në nivelin tjetër.

Nga ana matematikore, BFS përdor një **radhë** (queue) për të mbajtur gjurmët e nyjeve të vizituara dhe të paeksploruara. Ai përfundon kur të gjitha nyjet janë vizituar ose kur nyja e kërkuar është gjetur.

**Shembull Matematikor:**
Në një graf \( G = (V, E) \), ku:
- \( V \) është një bashkësi nyjesh (vertex).
- \( E \) është një bashkësi lidhjesh (edges),
BFS eksploron nyjet në një renditje \( L \), ku \( L \) është ndarë sipas nivelit të grafit.

**Vizualizim:**
![Shembull i BFS](https://upload.wikimedia.org/wikipedia/commons/4/46/Animated_BFS.gif)

#### Kërkimi në Thellësi (DFS)
DFS (Depth-First Search) është një algoritëm që eksploron grafët duke shkuar sa më thellë të jetë e mundur në çdo degë para se të kthehet prapa.

Nga ana matematikore, DFS përdor një **stivë** (stack), qoftë përmes rekursionit ose strukturave eksplisite të të dhënave, për të mbajtur gjurmët e nyjeve të eksploruara. Ky algoritëm ka aplikime të shumta, duke përfshirë ndarjen e grafit në komponentë të lidhur dhe analizën e cikleve.

**Shembull Matematikor:**
Në një graf \( G = (V, E) \), DFS krijon një pemë të gjurmimit \( T \), ku çdo nyjë është vizituar vetëm një herë, dhe \( T \) është një pjesë e grafit fillestar \( G \).

**Vizualizim:**
![Shembull i DFS](https://upload.wikimedia.org/wikipedia/commons/7/7f/Depth-First-Search.gif)

### Rëndësia Matematikore
Të dy algoritmet janë të rëndësishëm në teorinë e grafëve dhe kanë aplikime të shumta në:
- Zgjidhjen e ekuacioneve diferenciale.
- Modelet e rrjedhës së rrjeteve.
- Problemet e optimizimit.

## Karakteristikat e Projektit
- **Vizualizime Interaktive**: Tregon qartë mënyrën se si funksionojnë BFS dhe DFS.
- **Mbështetje për Ndërfaqen në Gjuhën Shqipe**.
- **I aksesueshëm online**: Nuk ka nevojë për konfigurime të mëtejshme lokale.

## Si të Përdorni?
1. Hapni [ubt-matematike.netlify.app](https://ubt-matematike.netlify.app/).
2. Zgjidhni algoritmin që dëshironi të vizualizoni (BFS ose DFS).
3. Ndiqni hap pas hapi procesin e eksplorimit në graf.

## Referencat
- **H. Cormen, T. Leiserson, C. Rivest, and C. Stein**. *Introduction to Algorithms*, 3rd Edition. MIT Press, 2009.
- **J. Gross and J. Yellen**. *Graph Theory and Its Applications*, 2nd Edition. Chapman & Hall/CRC, 2005.
- [Wikipedia: Breadth-First Search](https://en.wikipedia.org/wiki/Breadth-first_search)
- [Wikipedia: Depth-First Search](https://en.wikipedia.org/wiki/Depth-first_search)

---
*Ky projekt është zhvilluar për të ndihmuar në kuptimin e koncepteve bazë të teorisë së grafëve dhe aplikimeve të tyre praktike.*
