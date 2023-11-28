# PHP

Ajout du fichiers stub.php dans le dossier **misc** pour l'intellisence

## Structure du dosiser modules/php

Dans le dossier modules, créer l'arborescence suivante

```text
modules
|- php
   |- Core
      |- Game.php
      |- Globals.php
      |- Notifications.php
   |- Traits
      |- Action.php
      |- Args.php
      |- States.php
   |- constants.inc.php
```

### Game.php

Ce fichier va servir à l'accès à la classe principale (table) ainsi qu'aux options de jeu

```php
<?php

namespace Cannonades\Core;

class Game {
    public static function get() {
        // TODO replace by the class name on your <projectname>.game.php
        return \cannonadesmg::get();
    }
}
```

### Globals.php

Ce fichier va servir à l'accès aux variables globales

```php
<?php

namespace Cannonades\Core;

class Globals extends \APP_DbObject {

    /*************************
     **** GENERIC METHODS ****
     *************************/

    private static function set(string $name, /*object|array*/ $obj) {
        $jsonObj = json_encode($obj);
        self::DbQuery("INSERT INTO `global_variables`(`name`, `value`)  VALUES ('$name', '$jsonObj') ON DUPLICATE KEY UPDATE `value` = '$jsonObj'");
    }

    public static function get(string $name, $asArray = null) {
        /** @var string */
        $json_obj = self::getUniqueValueFromDB("SELECT `value` FROM `global_variables` where `name` = '$name'");
        if ($json_obj) {
            $object = json_decode($json_obj, $asArray);
            return $object;
        } else {
            return null;
        }
    }
}
```

### Notifications.php

Ce fichier va servir à l'implémentation des méthodes pour envoyer des notifications à l'interface utilisateur

```php
<?php

namespace Cannonades\Core;

class Notifications extends \APP_DbObject {

    /*************************
     **** GENERIC METHODS ****
     *************************/

    private static function notifyAll($name, $msg, $args = [], $exclude_player_id = null) {
        if ($exclude_player_id != null) {
            $args['excluded_player_id'] = $exclude_player_id;
        }
        Game::get()->notifyAllPlayers($name, $msg, $args);
    }

    private static function notify($player_id, $name, $msg, $args = []) {
        Game::get()->notifyPlayer($player_id, $name, $msg, $args);
    }

    private static function message($msg, $args = [], $exclude_player_id = null) {
        self::notifyAll('message', $msg, $args);
    }

    private static function messageTo($player_id, $msg, $args = []) {
        self::notify($player_id, 'message', $msg, $args);
    }

    private static function getPlayerName(int $player_id) {
        return Game::get()->getPlayerNameById($player_id);
    }

}
```

### Actions.php

Ce fichier va servir à l'implémentation des méthodes d'actions utilisateurs

```php
<?php

namespace Cannonades\Traits;

trait Actions {

    //////////////////////////////////////////////////////////////////////////////
    //////////// Player actions
    ////////////

    /*
        Each time a player is doing some game action, one of the methods below is called.
        (note: each method below must match an input method in cannonadesmg.action.php)
    */
}
```

### Args.php

Ce fichier va servir à l'implémentation des méthodes pour les arguments des states

```php
<?php

namespace Cannonades\Traits;

trait Args {

    //////////////////////////////////////////////////////////////////////////////
    //////////// Game state arguments
    ////////////

    /*
        Here, you can create methods defined as "game state arguments" (see "args" property in states.inc.php).
        These methods function is to return some additional information that is specific to the current
        game state.
    */
}
```

### States.php

Ce fichier va servir à l'implémentation des méthodes pour les states

```php
<?php

namespace Cannonades\Traits;

trait States {

    //////////////////////////////////////////////////////////////////////////////
    //////////// Game state actions
    ////////////

    /*
        Here, you can create methods defined as "game state actions" (see "action" property in states.inc.php).
        The action method of state X is called everytime the current game state is set to X.
    */
}
```

### constants.inc.php

Ce fichier va servir à l'implémentation des méthodes pour les states

```php
<?php

/**
 * States
 */
const ST_BGA_GAME_SETUP = 1;
const ST_END_GAME = 99;
```

## Altération de la classe principale du jeu

### cannonadesmg.game.php

```php
<?php

// ...

$swdNamespaceAutoload = function ($class) {
    $classParts = explode('\\', $class);
    if ($classParts[0] == 'Cannonades') {
        array_shift($classParts);
        $file = dirname(__FILE__) . '/modules/php/' . implode(DIRECTORY_SEPARATOR, $classParts) . '.php';
        if (file_exists($file)) {
            require_once $file;
        } else {
            var_dump('Cannot find file : ' . $file);
        }
    }
};
spl_autoload_register($swdNamespaceAutoload, true, true);

require_once(APP_GAMEMODULE_PATH . 'module/table/table.game.php');
require_once('modules/php/constants.inc.php');

class cannonadesmg extends Table {
    use Cannonades\Traits\Actions;
    use Cannonades\Traits\Args;
    use Cannonades\Traits\States;

    /** @var cannonadesmg */
    public static $instance = null;

    function __construct() {
        // ...

        self::$instance = $this;
    }

    public static function get() {
        return self::$instance;
    }

    // ...

    // TODO : Remove those sections
    // - Utility functions
    // - Player actions
    // - Game state arguments
    // - Game state actions
}
```

## Mise en place de la base de données

### dbmodel.sql

```sql
-- ------
-- BGA framework: © Gregory Isabelli <gisabelli@boardgamearena.com> & Emmanuel Colin <ecolin@boardgamearena.com>
-- cannonadesmg implementation : © <Your name here> <Your email address here>
-- 
-- This code has been produced on the BGA studio platform for use on http://boardgamearena.com.
-- See http://en.boardgamearena.com/#!doc/Studio for more information.
-- -----

-- dbmodel.sql

-- This is the file where you are describing the database schema of your game
-- Basically, you just have to export from PhpMyAdmin your table structure and copy/paste
-- this export here.
-- Note that the database itself and the standard tables ("global", "stats", "gamelog" and "player") are
-- already created and must not be created here

-- Note: The database schema is created from this file when the game starts. If you modify this file,
--       you have to restart a game to see your changes in database.

CREATE TABLE IF NOT EXISTS `card` (
  `card_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `card_type` varchar(16) NOT NULL,
  `card_type_arg` int(11) NOT NULL,
  `card_location` varchar(16) NOT NULL,
  `card_location_arg` int(11) NOT NULL,
  PRIMARY KEY (`card_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 AUTO_INCREMENT=1 ;

CREATE TABLE IF NOT EXISTS `global_variables` (
  `name` varchar(255) NOT NULL,
  `value` JSON,
  PRIMARY KEY (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

### Vérification des modifications

Démarrer une partie à 2 joueurs. Vous devriez voir ceci:

![Page initiale](../img/03/initial-game-page.png)
