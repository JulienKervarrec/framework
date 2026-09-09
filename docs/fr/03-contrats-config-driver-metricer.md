# Chapitre 3 -- Config, Driver, Metricer : les contrats que le service consomme

Trois interfaces completent le paquet `service/`, chacune representant un
point d extension que l implementation concrete (`example/`) doit
satisfaire.

`Driver` est volontairement le plus petit contrat possible : deux methodes,
`Start(ctx) error` et `Stop(ctx) error` -- aucune hypothese n est faite sur
ce que fait reellement le service, seulement qu il peut demarrer et
s arreter proprement. C est cette minimalite qui rend le framework
reutilisable pour a peu pres n importe quel service de fond (un indexeur,
un relayeur, un moniteur), le driver de l exemple (chapitre 4) n etant
qu une des implementations possibles.

`Metricer` combine `opmetrics.RegistryMetricer` (l interface standard
op-service pour exposer un registre Prometheus) avec deux methodes propres
au framework, `RecordInfo(version)` et `RecordUp()`, que `Service.Start`
appelle explicitement une fois le demarrage termine -- garantissant que
tout service construit sur ce framework expose au minimum ces deux
metriques de sante de base, independamment de ce que le service fait par
ailleurs.

`Config` regroupe quatre sous-configurations, chacune lue depuis les
arguments de ligne de commande via les lecteurs standard d op-service :
`LogConfig` (`oplog.CLIConfig`), `MetricsConfig` (`opmetrics.CLIConfig`),
`PprofConfig` (`oppprof.CLIConfig`), `RPCConfig` (`oprpc.CLIConfig`).
`NewConfig(ctx *cli.Context)` construit ces quatre sous-configurations en
un seul appel a partir du contexte `urfave/cli`, et `Check()` valide
sequentiellement les trois configurations qui exposent une methode de
validation (metriques, pprof, RPC) -- centralisant la validation de tous
les flags de ligne de commande avant que le service ne tente de demarrer
quoi que ce soit.
