# Mise en place du UI

## Mise en place des composantes

Afin de garder une certaine granularité dans nos fichiers, nous allons séparer le jeu en plusieurs composantes. Ainsi, il sera plus facile de savoir quoi mettre comme code dans chacun des fichiers.

**Note :** Vous pouvez décider de tout mettre dans le fichier principale *cannonades.ts* mais je vous le déconseille. Le fichier va devenir volumineux très rapidement et il sera difficile de garder le code bien organisé.

### Structure des fichiers à ajouté / modifié

```text
src/
  |- _player-table.scss
  |- _table-center.scss
  |- cannonades.d.ts (modify)
  |- cannonades.scss (modify)
  |- cannonades.ts (modify)
  |- player-table.ts
  |- table-center.ts
cannonadesmg_cannonadesmg.tpl (modify)
tsconfig.json (modify)
```

#### cannonades.scss

```scss
@import 'player-table';
@import 'table-center';
```

#### tsconfig.json

```json
    "files": [
        "src/bga-framework.d.ts",
        
        "src/player-table.ts",
        "src/table-center.ts",

        "src/cannonades.d.ts",
        "src/cannonades.ts",
        "src/define.ts"
    ]

```

#### player-table.ts

```ts
class PlayerTable {
    public player_id: number;

    constructor(public game: Cannonades, player: CannonadesPlayerData) {
        this.player_id = Number(player.id);

        const html = `<div id="player-table-${player.id}" class="player-table whiteblock" style="--player-color: #${player.color}">
            <h3>${player.name}</h3>
        </div>`;

        document.getElementById("tables").insertAdjacentHTML("beforeend", html);
    }
}
```

#### table-center.ts

```ts
class TableCenter {
    constructor(public game: Cannonades) {
    }
}
```

#### cannonades.ts

```ts
class Cannonades implements ebg.core.gamegui {
    public playersTables: PlayerTable[];

    public setup(gamedatas: CannonadesGamedatas) {
        this.createPlayerTables(gamedatas);
        this.setupNotifications();
    }
    public onEnteringState(stateName: string, args: any) {}
    public onLeavingState(stateName: string) {}
    public onUpdateActionButtons(stateName: string, args: any) {}
    public setupNotifications() {}

    private createPlayerTables(gamedatas: CannonadesGamedatas) {
        this.playersTables = [];
        gamedatas.playerorder.forEach((player_id) => {
           const player = gamedatas.players[Number(player_id)];
           const table = new PlayerTable(this, player);
           this.playersTables.push(table);
        });
     }
}
```

### cannonadesmg_cannonadesmg.tpl

```tpl
{OVERALL_GAME_HEADER}

<div id="table">
    <div id="tables-and-center">
        <div id="table-center-wrapper">
            <div id="table-center"></div>
        </div>
        <div id="tables"></div>
    </div>
</div>

{OVERALL_GAME_FOOTER}
```

### _player-table.scss

```scss
.player-table {
    h3 {
        color: var(--player-color);
    }
}
```

### _table-center.scss

```scss
#table-center-wrapper {
  display: flex;
  justify-content: center;
}

#table-center {
}
```