# Chapitre 4 -- L exemple : driver, API RPC et branchement CLI

`example/` illustre les trois contrats du chapitre 3 avec un service
concret : un `driver` qui interroge periodiquement la derniere entete de
bloc L1 via un `ethclient.Client`, et une API JSON-RPC qui expose cette
donnee ainsi qu un point d entree pour soumettre des transactions.

Le `driver` concret embarque un `txmgr.TxManager` (le gestionnaire de
transactions standard d op-service, qui gere retry et bump de gas), un
client `ethclient` connecte a `cfg.L1EthRpc`, et une boucle interne
(`loop`) lancee dans une goroutine au demarrage : un `time.Ticker` cadence
par `cfg.PollInterval` declenche a chaque tick un appel
`HeaderByNumber(ctx, nil)` pour recuperer la derniere entete, stockee dans
`d.latest` et enregistree comme metrique via `m.RecordL1Ref`. `Stop`
ferme le `txManager`, annule le contexte d arret, et attend
(`wg.Wait()`) que la goroutine de boucle se termine proprement avant de
retourner -- un patron d arret gracieux standard en Go pour une boucle de
fond.

`example/api/api.go` expose une API RPC sous l espace de noms `example`
(`rpc.API{Namespace: "example", Service: a}`) avec deux methodes : `
LatestHeader()` (retourne l entete la plus recente vue par le driver) et
`SendTx(ctx, candidate txmgr.TxCandidate)` (delegue directement a
`d.TxManager().Send`, exposant ainsi le gestionnaire de transactions du
driver via RPC) -- chaque appel enregistrant sa duree via
`m.RecordRPCServerRequest`, le patron d instrumentation systematique de ce
depot.

`example/cmd/main.go` assemble le tout dans une application CLI
`urfave/cli` : `oplog.SetupDefaults()`, construction de `cli.App` avec des
flags proteges (`cliapp.ProtectFlags`), une version formatee injectee par
le Makefile (`Version`, `GitCommit`, `GitDate`, des variables vides
peuplees a la compilation), et une fonction `Main` qui construit `Config`,
le `Logger`, le `Service`, le `Metricer`, le `Driver`, les enregistre via
le style fluide `WithMetrics`/`WithDriver`/`WithRPC` du chapitre 2, et
retourne le `Service` compose a `cliapp.LifecycleCmd`, qui prend en charge
l execution et l arret sur signal (`ctxinterrupt.WithSignalWaiterMain`).
