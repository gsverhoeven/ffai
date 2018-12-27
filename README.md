# FFAI: Fantasy Football AI
A highly-extensible modular multi-purpose AI framework for digital fantasy-football board-games.
FFAI is still under development and will be updated heavily the next six months.

## Features
* Rule implementation of the Living Rulebook 5 with the following limitations:
  * Only skills for the Human and Orc teams have been implemented and tested
  * No big guys
  * No league or tournament play
  * No star player points or level up
  * No inducements
  * No timers; Players have unlimited time each turn
  * Only premade teams
* A web interface supporting:
  * Hot-seat 
  * Online play
  * Spectators
  * Human vs. bot
* An AI interface that allows you to implement and test your own bots
* Implementation of the Open AI Gym interface, that allows you to train machine learning algorithms
* Custom pitches (we call these _arenas_). FFAI comes with arenas of four different sizes.
* Rule configurations are possible from a configuration file, including:
  * Arena (which .txt file to load describing the arena)
  * Ruleset (which .xml file to load containing rules for rosters etc.)
  * Setup restrictions
  * Number of turns
  * Kick-off table enabled/disabled
  * Which scatter dice to use
  * ...
* Premade formations to ease the setup phase. Custom made fomrations can easily be implemented. 
* Games can be saved and loaded

## Plans for Future Releases
* More documentation
* AI tournament module
* Dungeon arenas rules
* Support for all skills and teams in LRB6
* League mode
* Integration with OBBLM

## Installation
Clone the repository and make sure python 3.6 or newer is installed, together with pip.
Go to the cloned directory and run the following to install the dependencies: 
```
pip install -e .
```
Or
```
pip3 install -e .
```
Depending on your setup.

## Run FFAI's Web Server
```
python ffai/web/server.py
```
Go to: http://127.0.0.1:5000/

The home page lists active games. For each active game, you can click on a team to play it. If a team is disabled it is a bot and cannot be selected. Click hot-seat to play human vs. human on the same machine.

## Create a Bot
To make you own bot you must implement the Agent class and it's three methods: new_game, act, and end_game which are called by the framework. The act method must return an instance of the Action class. 

Take a look at our example here.

Be aware, that you should not modify instances that comes from the framework such as Square instances as these are shared with the GameState instance in FFAI. In future releases, we plan to release an AI tournament module that will clone the instances before they are handed to the bots.

## Gym Interface
FFAI implements the Open AI Gym interace for easy integration of machine learning algorithms. 

Take a look at our example here.

### Observations
Observations are split in two parts, one that consists of (spatial) two-dimensional feature leayers and a non-spatial vector of normalized values (e.g. turn number, half, scores etc.).

The default layers are these:
```
0. OccupiedLayer()
1. OwnPlayerLayer()
2. OppPlayerLayer()
3. OwnTackleZoneLayer()
4. OppTackleZoneLayer()
5. UpLayer()
6. UsedLayer()
7. AvailablePlayerLayer()
8. AvailablePositionLayer()
9. RollProbabilityLayer()
1. BlockDiceLayer()
10. ActivePlayerLayer()
12. MALayer()
13. STLayer()
14. AGLayer()
15. AVLayer()
16. MovemenLeftLayer()
17. BallLayer()
18. OwnHalfLayer()
19. OwnTouchdownLayer()
20. OppTouchdownLayer()
21. SkillLayer(Skill.BLOCK)
22. SkillLayer(Skill.DODGE)
23. SkillLayer(Skill.SURE_HANDS)
24. SkillLayer(Skill.PASS)
25. SkillLayer(Skill.BLOCK)
```
Custom layers can be implemented like this:
```
from ffai.ai import FeatureLayer
class MyCustomLayer(FeatureLayer):

    def produce(self, game):
        out = np.zeros((game.arena.height, game.arena.width))
        for y in range(len(game.state.pitch.board)):
            for x in range(len(game.state.pitch.board[0])):
                player = game.state.pitch.board[y][x]
                out[y][x] = 1.0 if player is not None and player.role.cost > 80000 else 0.0
        return out

    def name(self):
        return "expensive players"
```
And then added to the environmnet's feature layers:
```
env.layers.append(MyCustomLayer())
```

To visualize the feature layers, use the feature_layers option when calling render:
```
env.render(feature_layers=True)
```

### Action Types
Actions consists of 31 action types. Some action types, denoted by <position> also requires an x and y-coordinate.

```
0. ActionType.START_GAME
1. ActionType.HEADS
2. ActionType.TAILS
3. ActionType.KICK
4. ActionType.RECEIVE
5. ActionType.END_SETUP
6. ActionType.END_PLAYER_TURN
7. ActionType.USE_REROLL
8. ActionType.DONT_USE_REROLL
9. ActionType.END_TURN
10. ActionType.STAND_UP
11. ActionType.SELECT_ATTACKER_DOWN
12. ActionType.SELECT_BOTH_DOWN
13. ActionType.SELECT_PUSH
14. ActionType.SELECT_DEFENDER_STUMBLES
15. ActionType.SELECT_DEFENDER_DOWN
16. ActionType.SELECT_NONE
17. ActionType.SETUP_FORMATION_WEDGE
18. ActionType.SETUP_FORMATION_LINE
19. ActionType.SETUP_FORMATION_SPREAD
20. ActionType.SETUP_FORMATION_ZONE
21. ActionType.PLACE_PLAYER<Position>
22. ActionType.PLACE_BALL<Position>
23. ActionType.SELECT_SQUARE<Position>
24. ActionType.INTERCEPTION<Position>
25. ActionType.SELECT_PLAYER<Position>
26. ActionType.MOVE<Position>
27. ActionType.BLOCK<Position>
28. ActionType.PASS<Position>
29. ActionType.FOUL<Position>
30. ActionType.SELECT_SQUARE<Position>
31. ActionType.HANDOFF<Position>
```

Actions can be instantiated and used like this:
```
action = {
    'action-type': 26,
    'x': 8,
    'y': 6
}
obs, reward, done, info = env.step(action)
```

### Rewards and Info
The default reward function only rewards for a win, draw or loss 1/0/-1.
However, the info object returned by the step function contains useful information for reward shaping:
```
'cas_inflicted': {int},
'opp_cas_inflicted': {int},
'touchdowns': {int},
'opp_touchdowns': {int},
'half': {int},
'round': {int},
```
These values are commulative, such that 'cas_inflicted' referes to the total number of casualties inflicted by the team.

### Disclaminers and Copyrighted Art
FFAI is not affiliated with or endoresed by any company and/or trademark. FAAI is an open research framework and the authors have no commercial interests in this project. The web interface in FFAI currently uses a small set of icons from the Fantasy Football Client. These icons are not included in the license of FFAI. If you are the author of these icons and don't want us to use them in this project, please contact us at njustesen at gmail dot com, and we will replace them ASAP.