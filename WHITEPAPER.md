# Droid (DRX) Whitepaper

## Úvod

Projekt Droid vznikl z přesvědčení, že technologická bariéra (znalost programování) by neměla bránit realizaci inovativních ekonomických myšlenek. Od léta 2025 do 1. ledna 2026 byl DRX vyvíjen unikátní metodou: zakladatel, vystupující jako DarkwalkerPrime, definoval architekturu a pravidla sítě, zatímco samotnou implementaci kódu realizovali dva pokročilí AI asistenti – Google Gemini a Grok.

Tento proces nebyl pouhým generováním kódu, ale intenzivním iterativním vývojem. DarkwalkerPrime řídil logiku konsensu, ekonomické parametry a bezpečnostní protokoly, zatímco AI nástroje sloužily jako expertní „programátorské oddělení“.

 * Role AI: Převod vizí do stabilního kódu v Pythonu, implementace kryptografie SHA3-256 a správa P2P síťové vrstvy.

 * Role člověka: Strategické řízení, testování, definování měnové politiky (halving, celková zásoba) a finální validace bezpečnosti.

Droid (DRX) je důkazem nové éry vývoje softwaru. Ukazuje, že v roce 2026 může jednotlivec s jasnou vizí do určité míry konkurovat celým vývojářským týmům. DRX není jen měnou, je oslavou spolupráce lidské kreativity a umělé inteligence.

# Technické Specifikace Kryptoměny Droid (DRX)

Následující dokument popisuje technické specifikace kryptoměny Droid (ticker: DRX) na základě zdrojového kódu. Specifikace zahrnují architekturu, kryptografické prvky, parametry blockchainu, transakce, síťovou komunikaci a další klíčové vlastnosti. Kryptoměna je implementována v Pythonu s využitím knihoven jako ECDSA, hashlib a cryptography. Je navržena jako decentralizovaná síť s Proof-of-Work (PoW) konsenzem, inspirovaná Bitcoinem, ale s vlastními úpravami pro bezpečnost a efektivitu.

## 1. Základní Informace
- **Název projektu**: Droid
- **Ticker symbol**: DRX
- **Desetinná místa (decimals)**: 8 (např. 1 DRX = 10^8 jednotek)
- **Maximální nabídka (max supply)**: 100 000 000 DRX (100 000 000 * 10^8 jednotek)
- **Genesis blok**:
  - Timestamp: 1767297600 (odpovídá 1.1.2026, 20:00:00 UTC)
  - Genesis adresa: DRXfc3d428153b2c71be82e84a04c8b70b3d5153c75cc7c75edc3323d6e9c7cb8d5d063
  - Očekávaný hash genesis adresy: be2a18b752ca5804e83a2e2b2fd182654284d95a1bb51893ce5bcdf8f199bd42
  - Očekávaný hash genesis bloku: 00000b96e6ed7211e538e047307f7a5df3f07863bf1c4be9e1f6ea2b1eac8525
  - Coinbase odměna: 50 DRX (50 * 10^8 jednotek)
  - Data v coinbase transakci: "BTC: 00000000000000000000c6f7f5be25ea8f1f034b3bc7cf3c152f93289ebb33e0" (odkaz na Bitcoin blok)
- **Read-only režim**: Pokud NTP servery nejsou dostupné, systém běží v read-only režimu (bez těžby nebo transakcí).

## 2. Konsenzus a Těžba
- **Konsenzus mechanismus**: Proof-of-Work (PoW) s SHA3-256 hashovacím algoritmem.
- **Počáteční obtížnost (difficulty)**: 20 bitů (fixed target: (1 << 256) >> 20).
- **Dynamická úprava obtížnosti**: Ano (DYNAMIC_DIFFICULTY_ADJUSTMENT = True).
  - Interval úpravy: Každých 10 bloků (DIFFICULTY_ADJUSTMENT_INTERVAL).
  - Cílový čas na interval: 600 sekund (TARGET_BLOCK_TIME = 60 s * 10).
  - Úprava: Nový target = starý target * (skutečný čas / cílový čas), omezený na [1, (1 << 256) - 1].
- **Čas bloku**: Cílový 60 sekund (BLOCK_TIME_SECONDS).
- **Odměna za blok**: 50 DRX (50 * 10^8 jednotek), zahrnuje poplatky z transakcí.
- **Halving interval**: Každých 1 000 000 bloků (HALVING_INTERVAL_BLOCKS), odměna se dělí 2.
- **Těžba**:
  - Podporuje vícejádrový mining (uživatel volí počet CPU jader, max = multiprocessing.cpu_count()).
  - Reset nonce při změně timestampu (každých 60 sekund).
  - Žádné prázdné bloky povoleny (ALLOW_EMPTY_BLOCKS = True, ale kontrola vyžaduje alespoň jednu uživatelskou transakci, pokud ALLOW_EMPTY_BLOCKS = False).
  - Maximální velikost bloku: 1 MB (MAX_BLOCK_SIZE_BYTES).
- **Potvrzení**: Práh pro bezpečné potvrzení: 6 bloků (CONFIRMATIONS_THRESHOLD).

## 3. Kryptografie a Bezpečnost
- **Křivka pro klíče**: SECP256k1 (ECDSA s hashfunc=SHA3-256).
- **Generování adres**:
  - Adresa = TICKER + SHA3-256(public_key_bytes) + checksum (první 4 znaky SHA3-256(base_address)).
  - Délka adresy: 71 znaků (s checksum) nebo 67 (bez, pro zpětnou kompatibilitu).
- **Podpis transakcí**: ECDSA s SHA3-256, zpráva je JSON transakce (bez public_key a signature).
- **Hashování**:
  - Transakce a bloky: SHA3-256.
  - Merkle root: SHA3-256 kombinovaných TX ID (rekurzivní párování, duplikování posledního při lichém počtu).
- **Šifrování peněženek**: AES-GCM s PBKDF2 (SHA256, 100 000 iterací, salt 16 bajtů, nonce 12 bajtů).
- **Časová synchronizace**: NTP servery (pool.ntp.org, time.nist.gov, time.google.com), offset aplikován na time.time().
- **Ochrana proti útokům**:
  - Rate limiting: 10 požadavků/sekundu/IP (RATE_LIMIT_REQUESTS), okno 1 s.
  - TX rate limiting: 100 TX/60 s/IP.
  - Maximální velikost zprávy: 10 MB.
  - Blacklist pro DoS.
  - Maximální počet peerů: 20 (MAX_PEERS) pro ochranu proti Sybil.
  - Validace nonce: Vzestupné, bez duplicit v bloku/chainu/mempoolu.
  - Orphan pool pro řešení forků, recyklace TX z osiřelých bloků.

## 4. Transakce
- **Struktura transakce**:
  - Pole: from_address, to_address, amount, fee, nonce, timestamp, public_key, signature, tx_id (SHA3-256 JSON), data (pouze v coinbase).
- **Coinbase transakce**: Pouze jedna na blok, první v seznamu, odměna + poplatky.
- **Minimální částka**: 0.00000001 DRX (MIN_TX_AMOUNT).
- **Poplatky**: Min 0.00000001 DRX, max 0.01 DRX (TX_FEE_MIN/MAX).
- **Nonce**: Sekvenční, pro prevenci replay attacks; očekávaná = max_nonce + 1 + pending_count.
- **Validace**:
  - Žádný transfer na stejnou adresu.
  - Platný timestamp (do +600 s).
  - Podpis a identita odesílatele.
  - Dostatečný zůstatek (potvrzený - pending výstupy).
  - Bez duplicit TX ID/nonce v mempoolu/chainu.
- **Mempool**:
  - Maximální velikost: 10 MB (MAX_MEMPOOL_SIZE_BYTES).
  - Uložení v SQLite (MEMPOOL_DB).
  - Výběr TX do bloku: Priorita podle poplatku (heapq), seřazení nonce pro odesílatele.
- **Potvrzení**: TX je potvrzena po zařazení do bloku a dostatečných potvrzeních.

## 5. Blockchain Struktura
- **Uložení**: SQLite databáze (BLOCKCHAIN_DB pro bloky, MEMPOOL_DB pro mempool).
  - Bloky: Pouze posledních 100 v RAM (LAST_BLOCKS_TO_KEEP).
  - Stav: balance_map, nonce_map, all_tx_ids (rebuild z DB při potřebě).
- **Blok struktura**:
  - Pole: index, timestamp, transactions, merkle_root, previous_hash, target (hex), nonce, hash (SHA3-256 JSON).
- **Validace chainu**:
  - Kontrola genesis, hashů, targetů, timestampů, nonce, duplicit, zůstatků, max supply.
  - Nahrazení chainu: Pouze pokud nový má vyšší kumulativní práci (work = (1 << 256) // target) nebo stejnou práci a delší délku/tie-breaker (menší last_hash).
  - Žádné přepsání bloků s >=6 potvrzeními.
- **Fork řešení**: Orphan pool, rekurzní přidávání po rodiči, recyklace TX.

## 6. Síťová Komunikace (P2P)
- **Host a port**: 0.0.0.0, výchozí 5001 (lze specifikovat při spuštění).
- **Zprávy**: JSON přes TCP, prefix 4 bajty délky zprávy.
  - Typy: transaction, new_block, request_chain_info, response_chain_info, request_blocks, response_blocks, request_full_chain, response_full_chain, request_mempool, response_mempool, new_peer.
- **Synchronizace**: Periodická (každých 10 s), požadavek info, bloků nebo full chain.
- **Peers**: Uložení v JSON (PEERS_FILE), automatické připojení, sdílení mempoolu a chainu.
- **Bezpečnost**: Timeouty, rate limiting, blacklist, maximální velikost zpráv.

## 7. Další Funkce
- **Peněženky**: Uložené šifrovaně (WALLETS_FILE.enc), import/export privátních klíčů.
- **NTP synchronizace**: Ano, s fallback na read-only při selhání.
- **Logování**: P2P log v queue, barevný výstup s colorama.
- **Uživatelské rozhraní**: Konzolové menu pro těžbu, transakce, zobrazení atd.
- **Knihovny**: hashlib, json, time, ecdsa, binascii, os, threading, socket, sys, queue, colorama, struct, cryptography, getpass, sqlite3, heapq, multiprocessing, collections, readline, math, signal.

## Roadmap — stav vývoje a přechod na finální verzi

V současné fázi (Alpha 0.1.0) je zdrojový kód veřejně dostupný, aby bylo možné rychlé testování, debugování a komunitní přispívání i na stolních počítačích. Toto otevřené vydání slouží výhradně jako raná testovací verze a může obsahovat neúplné funkce a bezpečnostní nedostatky.

## Plán pro finální verzi:

- Finální klient bude navržen jako Android/Termux‑only aplikace — implementována bude detekce platformy (ARM / Termux) tak, aby oficiální klient nebyl spustitelný na běžných PC nebo v emulátorech.  
- Bude zavedena integrační kontrola integrity klienta (např. kontrolní součet / checksum), která minimalizuje riziko používání upravených nebo kompromitovaných klientů při interakci s oficiální sítí.  
- Finální verze nebude zpětně kompatibilní s klienty alfa verze; blockchainová historie (ledger) však zůstane zachována, takže data a transakce z alfa fáze budou integrované do finální sítě.  
- Do finální sítě budou implementována opatření proti přijímání transakcí z neoficiálních forků a klonů, aby se zabránilo míchání provozu a záměně transakcí mezi odlišnými instancemi softwaru.

## Varování
I v současné Alfa 0.1.0 verzi je implementována základní ochrana proti transakcím s upravenými konstantami, takže blockchain odmítá transakce od uzlů, které by se snažily podvádět změnou konstant ve svém kódu.  
 
## Poznámka
V současné fázi je nutné používat VPN (ZeroTierOne), protože projekt zatím nemá implementované peer discovery, ani šifrování P2P komunikace. Alternativně můžete použít SSH.

## Poděkování
Děkuji komunitě a odkazu Satoshiho Nakamota za inspiraci. Droid (DRX) byl dokončen 1. 1. 2026 jako pocta principům férového startu a decentralizace."

Poděkování patří všem, kteří věří v otevřené finanční systémy. Uznání náleží i síti Bitcoin, v jejímž bloku 930423 je tento projekt navždy kryptograficky ukotven.

Děkuji modelům Google Gemini a Grok za stovky hodin iterativního vývoje, během nichž transformovali mé architektonické definice do reality.
 
​Zvláštní poděkování patří i uživateli Whitehouse za
jeho odborné konzultace v oblasti implementace cílové obtížnosti (target difficulty), a za poskytnutí klíčového fragmentu kódu pro monitoring sítě:

sys.stdout.write(f"Current target difficulty: {difficulty}\n")

​Tento úryvek posloužil jako základ pro implementaci komplexního monitorovacího rozhraní, které v reálném čase (každých 60 sekund) informuje těžaře o resetu PoW Nonce, aktuální obtížnosti (target) a časovém razítku bloku.

## Dodatek
​Jako tvůrce Droid (DRX) jsem se rozhodl zachovat privátní klíč k genesis coinbase odměně (50 DRX) pouze pro nouzové situace. Tyto mince zůstanou nedotknuté a budou použity výhradně s transparentním souhlasem komunity – například na kritické opravy sítě, bezpečnostní audity nebo komunitní projekty. Žádný osobní zisk. To zajišťuje férový start a dlouhodobou udržitelnost projektu.

## Upozornění
Tento projekt slouží především pro vzdělávání a hobby; nejde o produkt určený pro produkční prostředí.

## Copyright (c) 2026 DarkwalkerPrime
