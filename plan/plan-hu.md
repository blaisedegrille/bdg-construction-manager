# Felújítási projektvezető – termékkoncepció és fejlesztési terv

## 1. A termék célja

Egy asztali alkalmazás, amellyel több felújítási projekt tervezhető és követhető egy helyen. A felhasználó lássa, mi a következő teendő, ki és mikor végzi el, mennyibe kerül, hogyan halad a projekt, és hol találhatók a kapcsolódó dokumentumok.

Az alkalmazás központi eleme a munkafolyamat és annak története legyen. A Jira-szerű tábla hasznos nézet, de a termék ne váljon általános feladatkezelővé: a feladatok kapcsolódjanak a felújítás szakaszaihoz, költségvetéséhez, kivitelezőihez, mérföldköveihez és dokumentumaihoz.

## 2. Javasolt technológiai irány

### Elsődleges javaslat

- **Electron** asztali keretrendszer Windowsra, macOS-re és Linuxra.
- **React + TypeScript** felhasználói felület.
- **MUI** felületi komponensekhez és világos/sötét témához.
- **Redux Toolkit + RTK Query** alkalmazásállapothoz és API-adatok kezeléséhez/gyorsítótárazásához; **React Hook Form + Zod** űrlapokhoz és validáláshoz.
- **NestJS** moduláris backend/API, amely az üzleti logikát, jogosultságokat, adatkezelést és későbbi szinkronizálást kezeli.
- **PostgreSQL** adatbázis Docker konténerben, tartós volume-mal.
- **Docker Compose** a NestJS API, PostgreSQL és az opcionális fájltároló helyi vagy szerveroldali indításához.
- **Prisma** adatmodellezéshez és adatbázis-migrációkhoz.

### Architektúra és fontos döntés

Elsődleges felállásként az alkalmazás **kliens–szerver architektúrában** működjön: a React felület Electron asztali kliensben fut, a NestJS API és PostgreSQL pedig Dockerben. A konténerek helyben, a felhasználó gépén vagy egy elérhető szerveren is futtathatók. A kliens konfigurálható API-címhez kapcsolódjon. Ha Electronból fájlmegnyitás vagy egyéb operációs rendszer funkció kell, azt biztonságos preload/IPC rétegen keresztül intézze; a renderer ne kapjon közvetlen Node.js-hozzáférést.

Mivel a backend eleve központi API-ként fut, a felhasználókezelés, projektmegosztás és jogosultságok illeszkedjenek az alapmodellhez. A Docker Compose fejlesztői és saját üzemeltetésű (self-hosted) telepítést is támogasson. A felhőszolgáltatási üzemeltetés és automatikus frissítés később építhető rá. Offline mód nem része az első kiadásnak; ha fontos igény lesz, külön szinkronizálási terv szükséges.

### Alternatívák

- **Fastify önmagában**: kisebb, közvetlenebb API-réteg, de több architekturális konvenciót a projektre hagy. Akkor válasszuk, ha a NestJS túl nehéznek bizonyul.
- **ASP.NET Core**: jó alternatíva, ha a csapat C#-ban jártas vagy a Microsoft ökoszisztéma elsődleges. Ebben a TypeScript-központú projektben a NestJS egységesebb fejlesztési nyelvet ad.
- A React felület később böngészőben is futtatható, így az Electron kliens mellett webes kliens is kiadható.

**Backend-ajánlás:** maradjon a **NestJS + PostgreSQL + Prisma + Docker Compose**. A NestJS illik a TypeScript-alapú React/Electron stackhez, modulárisan rendezi a hitelesítést, projekteket, feladatokat, költségeket és dokumentumokat, és később böngészős klienst is kiszolgálhat. A React nem kíván különleges backendet: a kliens és a szerver HTTP API-n keresztül, külön folyamatként kapcsolódik.

**Ajánlott teljes stack:** Electron + React + TypeScript + Vite + MUI + Redux Toolkit/RTK Query + React Hook Form/Zod; Electron Forge a csomagoláshoz; NestJS REST API, PostgreSQL és Prisma Docker Compose-ban. Az RTK Query kezelje a szerveradatok gyorsítótárát és frissítését, a Redux slice-ok pedig a megosztott kliensállapotot (például bejelentkezés, aktív projekt, felületi beállítások). Az űrlapok átmeneti értékeit ne másoljuk globális store-ba. A Vite Forge-integrációját az induló prototípusban ellenőrizni kell, mert a Forge Vite plugin dokumentációja jelenleg kísérleti státuszt jelez. Kezdetben REST elegendő; valós idejű frissítés csak konkrét igény esetén kerüljön be.

## 3. Fő felhasználói folyamatok

1. A felhasználó létrehoz egy projektet, megadja a nevét, címét/helyszínét, típusát, tervezett kezdését és befejezését, valamint a teljes keretösszeget.
2. A projektet szakaszokra bontja (például tervezés, bontás, gépészet, villanyszerelés, burkolás, festés, átadás), vagy egy sablonból indul.
3. A szakaszokhoz feladatokat és mérföldköveket rögzít: felelős, határidő, státusz, függőségek, becsült idő és tervezett költség.
4. Munka közben állapotot frissít, megjegyzést, dátumot, tényleges időt és tényleges költséget rögzít, illetve fotót vagy dokumentumot csatol.
5. A projekt áttekintőjén összeveti a tervet a tényleges haladással és költéssel, és látja a késedelmes vagy költségtúllépés-közeli tételeket.
6. A lezárt projekt archiválható, de később is kereshető és visszanézhető.

## 4. Funkciók és nézetek

### Projektek

- Több projekt kezelése, létrehozás, archiválás és keresés.
- Projektadatlap: leírás, helyszín, dátumok, költségkeret, állapot, résztvevők.
- Projekt sablonok gyakori felújítási típusokhoz; saját sablon mentése későbbi fázisban.
- Projektváltó és legutóbb megnyitott projektek.

### Tervezés és munkafolyamatok

- Szakaszok és feladatok listanézete, valamint Kanban tábla (teendő, folyamatban, akadályozott, kész – állapotok testre szabhatók később).
- Feladat részletei: cím, leírás, státusz, prioritás, felelős, kezdés, határidő, becsült/tényleges idő, tervezett/tényleges költség.
- Mérföldkövek és feladatfüggőségek; naptár- vagy idővonalnézet későbbi bővítésként.
- Ellenőrzőlista, megjegyzések, aktivitási napló és fájlcsatolmányok.
- Szűrés státusz, felelős, szakasz és határidő szerint.

### Költségek

- Költségterv kategóriák és feladatok szerint (anyag, munkadíj, engedély, szállítás, egyéb).
- Tervezett összeg, vállalt/lekötött összeg és ténylegesen kifizetett összeg külön kezelése.
- Költségrekord dátummal, kategóriával, partnerrel, megjegyzéssel és opcionális feladatkapcsolattal.
- Keret és tényleges költés összevetése; különbség összegben és százalékban.
- Deviza és adókezelés kezdetben projekt szintű beállításokkal; részletes könyvelési funkció nem cél az első kiadásban.

### Idő és előrehaladás

- Tervezett és tényleges kezdés/befejezés, becsült és tényleges ráfordítás.
- Feladat- és szakaszszintű előrehaladás, mérföldkövek teljesülése.
- Egyszerű aktivitási napló, hogy látszódjon, mikor és mi változott.
- Később opcionális időmérő; az MVP-ben a tényleges idő kézi rögzítése elegendő.

### Dokumentumok és fotók

- Ajánlatok, számlák, szerződések, tervek és előtte/utána fotók csatolása.
- Fájlnév, típus, feltöltés dátuma, megjegyzés és kapcsolódó projekt/feladat/költség.
- Fájlok biztonságos, projektenként elkülönített tárolása Docker volume-on vagy S3-kompatibilis tárolóban; adatbázistól külön menthetően.
- Fájlméret-korlát, támogatott formátumok és törlési viselkedés meghatározása.
- A fájlok tartalmából történő automatikus adatkinyerés (például számlaösszeg) későbbi, opcionális funkció.

### Felhasználók és megjelenés

- Felhasználói fiókok, bejelentkezés és projekt szerepkörök (tulajdonos, szerkesztő, megtekintő).
- Világos/sötét téma, rendszerbeállítás követése, menüpontok: Áttekintés, Projektek, Feladatok, Költségek, Dokumentumok, Beállítások.
- Magyar nyelvű felület elsőként; a szövegek lokalizálható módon készüljenek.
- Alap akadálymentesség: billentyűzetes használat, jól látható fókusz, megfelelő kontraszt.

## 5. MVP – az első használható verzió határa

Az első kiadásban legyen:

- több projekt létrehozása, szerkesztése és archiválása;
- projekt áttekintő alapadatokkal és keretösszeggel;
- szakaszok, mérföldkövek, feladatok és Kanban/listanézet;
- státusz, határidő, felelős, becsült/tényleges idő és költség;
- költségrekordok és terv–tény összesítés;
- megjegyzések/aktivitási előzmények;
- dokumentumok csatolása, megnyitása és projekthez/feladathoz kapcsolása;
- világos és sötét téma, navigáció, alap keresés és szűrés;
- bejelentkezés, projekt tagság és alap szerepkörök;
- Docker Compose konfiguráció a NestJS API-val, PostgreSQL-lel és tartós adattárolással;
- adatbázis-migrációk és dokumentált mentés/visszaállítás.

MVP-n kívül maradjon a valós idejű együttműködés, mobilalkalmazás, részletes jogosultság-adminisztráció, számlafelismerés, beépített csevegés, Gantt kritikus útvonal, számlázás és külső könyvelői integráció. Ezekről az MVP kipróbálása után érdemes dönteni.

## 6. Fejlesztési ütemterv

Az időbecslések egy fejlesztői kapacitásra és a scope tisztázása után érvényes durva becslések; a technikai prototípus eredménye módosíthatja őket.

### 0. fázis – termék- és technikai alapozás (2–4 nap)

- Felhasználói történetek, MVP-határ és fogalmak rögzítése.
- A projekt, szakasz, feladat, mérföldkő, költség, dokumentum és felhasználó közötti kapcsolatok vázolása.
- UI vázlatok: projektlista, áttekintő, Kanban, feladat részletei, költségek, beállítások.
- Rövid döntési dokumentum az offline, mentés és későbbi szinkron elvárásairól.

**Kimenet:** elfogadható MVP-specifikáció és képernyővázlatok.

### 1. fázis – scaffolding és kockázatos technikai próba (3–5 nap)

- Repo felépítése, Electron Forge + Vite + React/TypeScript alapalkalmazás és MUI téma.
- Redux Toolkit store, RTK Query API réteg, React Hook Form és Zod alapkonfiguráció.
- NestJS moduláris backend és Electron preload/IPC/API kommunikáció.
- Docker Compose: NestJS API és PostgreSQL, környezeti konfigurációval és tartós volume-mal.
- Adatbázis-migráció, egészségellenőrzés, naplózás, API-kapcsolat az Electron kliensből.
- Fájlok tartós tárolásának prototípusa; mentési és visszaállítási folyamat vázolása.
- Teljes stack indítása/leállítása és csomagolt klienssel való kapcsolódás ellenőrzése.

**Kimenet:** induló kliens és Dockerben futó API/adatbázis, igazolt tartós adattárolással.

### 2. fázis – alkalmazásváz és projektek (1 hét)

- Fő navigáció, projektlista, projekt létrehozása/szerkesztése/archiválása.
- Alap adatmodell és CRUD műveletek.
- Világos/sötét téma és beállítások.
- Üres állapotok, hibakezelés, megerősítő párbeszédek.

**Kimenet:** több projektet kezelő, következetesen navigálható alkalmazás.

### 3. fázis – tervezés, feladatok és mérföldkövek (1–2 hét)

- Projekt szakaszok és feladatok kezelése.
- Lista- és Kanban-nézet, feladat létrehozása/szerkesztése, státuszváltás.
- Határidő, prioritás, becsült idő, felelős és függőségi alapok.
- Mérföldkövek és projekt aktivitási napló.

**Kimenet:** a projekt munkamenete elejétől a lezárásig követhető.

### 4. fázis – költségkövetés és időráfordítás (1 hét)

- Költségkategóriák és költségrekordok.
- Tervezett, vállalt és tényleges összegek rögzítése.
- Feladat becsült/tényleges idő és dátumok.
- Áttekintő kártyák és egyszerű terv–tény grafikonok/összesítések.

**Kimenet:** költség- és időeltérés jól látható projektenként.

### 5. fázis – dokumentumok és adatbiztonság (1 hét)

- Fájlcsatolás, előnézet/megnyitás, kapcsolás projekthez, feladathoz vagy költséghez.
- Docker volume vagy S3-kompatibilis tárhely és fájl-eltávolítás szabályai.
- Exportálható projektadat és mentés/visszaállítás.
- Helyi profil és egyszerű alkalmazászár, ha szükséges.

**Kimenet:** fontos ajánlatok, számlák és munkafotók a projekt mellett tárolhatók.

### 6. fázis – finomítás és első kiadás (3–5 nap)

- Üres, betöltési és hibaállapotok, keresés és szűrés.
- Magyar feliratok, billentyűzetes használat és témaellenőrzés.
- Telepítő készítése a célplatformokra, verziózás és frissítési stratégia.
- Felhasználói kipróbálás valós felújítási példával, majd javítások.

**Kimenet:** első használható, Dockerben üzemeltethető kliens–szerver kiadás.

### 7. fázis – kibővített együttműködés és üzemeltetés (külön tervezendő)

- Projektmeghívók, részletesebb szerepkörök és értesítések.
- Távoli elérés, TLS, titokkezelés és biztonságos telepítési útmutató.
- Opcionális S3-kompatibilis dokumentumtároló, automatikus mentés és visszaállítás.
- Felhőszolgáltatási üzemeltetés, monitorozás és frissítési folyamat.

**Kimenet:** megbízható, távolról is elérhető közös szolgáltatás.

## 7. Javasolt navigáció

- **Áttekintés** – összes projekt, közelgő határidők és friss aktivitás.
- **Projektek** – projektlista, létrehozás és archivált projektek.
- **Feladatok** – kiválasztott projekt feladatlistája/táblája.
- **Költségek** – költségkeret, kategóriák és tényadatok.
- **Dokumentumok** – projekt dokumentumainak kereshető listája.
- **Beállítások** – profil, téma, mentés, nyelv és alkalmazásbeállítások.

A projekt megnyitásakor a projekt saját almenüje tartalmazza az Áttekintés, Terv/Feladatok, Mérföldkövek, Költségek, Dokumentumok és Tevékenység lapokat.

## 8. Kezdeti adatmodell

- `User`: fiók, hitelesítési adatok hivatkozása, profil és állapot.
- `Project`: név, leírás, helyszín, státusz, dátumok, pénznem, költségkeret, létrehozó.
- `ProjectMember`: projekt, felhasználó, szerepkör (későbbi együttműködéshez).
- `Stage`: projekt, név, sorrend, tervezett időszak.
- `Task`: szakasz, cím/leírás, státusz, felelős, prioritás, határidők, idő- és költségterv/tény.
- `Milestone`: projekt, név, cél dátum, teljesülés.
- `TaskDependency`: feladat és előfeltétel kapcsolata.
- `CostEntry`: projekt/feladat, kategória, partner, összeg, pénznem, dátum, állapot.
- `Attachment`: fájl helye, neve, típusa, mérete és kapcsolódó rekord.
- `Activity`: felhasználó, művelet, időpont és érintett rekord.

## 9. Minőségi és működési elvárások

- PostgreSQL-adatvesztés elleni védelem, tartós Docker volume és egyértelmű mentési/visszaállítási folyamat.
- Stabil migrációk, hogy az alkalmazásfrissítés ne veszítse el a meglévő projekteket.
- A dokumentumok nem kerülhetnek véletlenül a projekt adatbázisába nagy bináris mezőként; fájlkezelésük külön történjen.
- Felhasználói fájlnevek és útvonalak biztonságos kezelése.
- Az API hitelesítést, jogosultság-ellenőrzést és projekt-adatelkülönítést alkalmazzon.
- A Docker image ne tartalmazzon titkokat vagy adatokat; konfiguráció és tartós állományok külön legyenek.
- Távoli elérés esetén HTTPS, biztonságos jelszókezelés és rendszeres biztonsági mentés szükséges.
- A csomagolt Electron alkalmazás biztonsági beállításai legyenek szigorúak: context isolation, sandbox, szűk IPC engedélyezés, tartalombiztonsági szabályzat és rendererben kikapcsolt Node integráció. Az IPC üzeneteket ellenőrizni kell, a preload csak célzott, minimális API-t tegyen elérhetővé.

## 10. Nyitott termékdöntések

Ezeket a prototípus és az első használati visszajelzés alapján kell véglegesíteni:

1. A Docker stack csak saját gépen fut majd, helyi hálózaton, vagy internet felől is elérhető szerveren?
2. Mely operációs rendszereket kell támogatni az első kiadásban?
3. A dokumentumok Docker volume-on vagy S3-kompatibilis tárolóban legyenek?
4. Milyen gyakran készüljön automatikus adatmentés, és hová kerüljön?
5. A pénznem és az adókezelés projekt szintű legyen-e?
6. Van-e olyan meglévő felújítási folyamat vagy táblázat, amelyből importálni kell?

**Javasolt alapfeltevés az induláshoz:** Docker Compose-szal indítható NestJS + PostgreSQL szolgáltatás, React/Electron kliens, több projekt és alap felhasználói fiókok. Az első kiadás egy gépen vagy saját szerveren üzemeltethető; az internet felőli elérés külön telepítési beállítás legyen.
