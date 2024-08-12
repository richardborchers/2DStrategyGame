<template>
    <v-container>
        <h1>Main game</h1>
        <v-btn @click=startPress()>Start</v-btn>
        <v-btn @click=skip()>Skip</v-btn>
        <v-row>
            <v-col cols="8">
                <v-card color="dark-grey">
                    <v-container>
                        <v-row no-gutters v-for="(row, rowIndex) in rows" :key="rowIndex">
                            <v-col v-for="(col, colIndex) in cols" :key="colIndex">
                                <v-hover>
                                    <template v-slot:default="{ isHovering, props }">
                                        <v-card v-bind="props" :color="computeColor(rowIndex, colIndex, isHovering)"
                                            @click="cellClick(rowIndex, colIndex)" class="grid-cell">
                                            <v-menu v-model="show" :close-on-content-click="false" location="end" offset-y>
                                                <!-- Renamed 'props' to 'menuProps' here to avoid conflict -->
                                                <template v-slot:activator="{ on, props: menuProps }">
                                                    <v-avatar class="vertical-center" size="40" v-bind="menuProps"
                                                        @click="cellClick(rowIndex, colIndex)">
                                                        <v-icon :color="board[rowIndex][colIndex].onTurn ? 'red' : 'black'"
                                                            :icon="board[rowIndex][colIndex].icon" size="50"></v-icon>
                                                    </v-avatar>
                                                </template>

                                                <v-card color="rgba(255, 255, 255, .3)" style="border: 1px solid black;"
                                                    variant="flat" elevation="0"
                                                    v-if="board[rowIndex][colIndex].onTurn && gameState.getCurrentState() === 'playerOptions'">
                                                    <v-card-text>
                                                        <div v-for="option in playerOptions" :key="option.text">
                                                            <v-chip :color="option.color" variant="flat"
                                                                :prepend-icon="option.icon" @click="option.event"
                                                                class="text-overline">
                                                                {{ option.text }}
                                                            </v-chip>
                                                        </div>
                                                    </v-card-text>
                                                </v-card>
                                                <v-card color="rgba(255, 255, 255, .3)" style="border: 1px solid black;"
                                                    variant="flat" elevation="0"
                                                    v-if="board[rowIndex][colIndex].onTurn && gameState.getCurrentState() === 'attackOption'">
                                                    <v-card-text>
                                                        <div v-for="option in board[rowIndex][colIndex].attackOptionList"
                                                            :key="option.text">
                                                            <v-chip variant="flat" color="black" :prepend-icon="option.icon"
                                                                class="text-overline" @click="attackOptionSelected(option)"  :disabled="board[rowIndex][colIndex].currentActionPoints < option.cost">
                                                                {{ option.name }}
                                                                <v-tooltip activator="parent" location="end">
                                                                    <v-card color="white" variant="flat">
                                                                        <v-card-text>
                                                                            <h4>Description</h4>
                                                                            {{ option.tooltipText }}
                                                                            <h4>Damage</h4>
                                                                            {{ option.damage }} ({{ option.damageType }})
                                                                            <h4>Range</h4>
                                                                            {{ option.rangeMin }} - {{ option.rangeMax }}
                                                                            <h4>Cost</h4>
                                                                            {{ option.cost }} point
                                                                        </v-card-text>
                                                                    </v-card>
                                                                </v-tooltip>
                                                            </v-chip>
                                                        </div>
                                                    </v-card-text>
                                                </v-card>
                                            </v-menu>
                                        </v-card>
                                    </template>
                                </v-hover>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-card>
            </v-col>
            <v-col cols="4">
                <v-card color="dark-grey">
                    <v-timeline side="end">
                        <v-timeline-item density="compact" v-for="turn in turnSystem" :key="turn.entity.id"
                            :dot-color="turn.entity.color" size="small">
                            <v-card :color="(turn.onTurn) ? 'green' : 'black'" :prepend-icon="turn.entity.icon"
                                :subtitle="turn.entity.health + '/' + turn.entity.maxHealth">
                                <template v-slot:title>
                                    <span class="font-weight-black">{{ turn.entity.text }}</span>
                                </template>
                                <v-progress-linear color="red" :model-value="turn.entity.health"
                                    :max="turn.entity.maxHealth"></v-progress-linear>
                            </v-card>
                        </v-timeline-item>
                    </v-timeline>
                </v-card>
            </v-col>
        </v-row>
    </v-container>
</template>

<script setup>
import { reactive, watch, ref } from 'vue';

const numRows = 8;
const rows = Array.from({ length: numRows });

const numCols = 8;
const cols = Array.from({ length: numCols });

const show = ref(true);

const playerAttackOptionList = [
    {
        name: "Stab",
        icon: 'mdi-knife-military',
        tooltipText: 'Attack an enemy from nearby',
        damage: 5,
        damageType: 'physical',
        cost: 1,
        rangeMin: 1,
        rangeMax: 1,
        AOE: 'single',
    },
    {
        name: "Sling",
        icon: 'mdi-target',
        tooltipText: 'Attack an enemy from a distance',
        damage: 5,
        damageType: 'piercing',
        cost: 1,
        rangeMin: 3,
        rangeMax: 7,
        AOE: 'single',
    }
]

const player = {
    player: true,
    occupied: true,
    selected: false,
    text: 'Jerry',
    icon: 'mdi-account-cowboy-hat',
    color: 'blue',
    health: 25,
    maxHealth: 50,
    maxMoveRange: 4,
    currentMoveRange: 4,
    currentActionPoints: 2,
    maxActionPoints: 2,
    onTurn: false,
    isEntity: true,
    attackOptionList: playerAttackOptionList,
}

const enemy = {
    player: false,
    occupied: true,
    selected: false,
    text: 'Monster',
    icon: 'mdi-bacteria',
    color: 'red',
    health: 44,
    maxHealth: 50,
    maxMoveRange: 4,
    currentMoveRange: 4,
    currentActionPoints: 2,
    maxActionPoints: 2,
    onTurn: false,
    isEntity: true,
    attackOptionList: playerAttackOptionList,
}

const empty = {
    player: false,
    occupied: true,
    selected: false,
    text: '',
    icon: '',
    color: '',
    health: 0,
    maxHealth: 0,
    onTurn: false,
    isEntity: false,
    attackOptionList: [],
}

const playerOptions = [{
    text: 'Attack',
    icon: 'mdi-sword-cross',
    event: (() => attackOption()),
    color: 'red'
},
{
    text: 'Move',
    icon: 'mdi-swap-horizontal',
    event: (() => moveOption()),
    color: 'white'
},
{
    text: 'Skip',
    icon: 'mdi-check',
    event: (() => skipOption()),
    color: 'green'
}
]

function testConsole() {
    console.log("test")
}

const selectedAttack = ref()

function attackOptionSelected(option) {
    gameState.transition('attackPlacement')
    showTargetableTiles(option)
    selectedAttack.value = option
}

function showTargetableTiles(attackOption) {
    let position = findObjectBoardPosition(turnSystem[0].entity);
    if (!position) {
        console.error("Object not found on the board");
    }

    let rowIndex = position.rowIndex;
    let colIndex = position.colIndex;
    let minRange = attackOption.rangeMin;
    let maxRange = attackOption.rangeMax;
    // Function to check if a position is within the board boundaries
    function isWithinBoard(row, col) {
        return row >= 0 && row < board.length && col >= 0 && col < board[0].length;
    }

    // Set tiles that are in range of obj.currentMoveRange to highlight
    for (let row = rowIndex - maxRange; row <= rowIndex + maxRange; row++) {
        for (let col = colIndex - maxRange; col <= colIndex + maxRange; col++) {
            if (isWithinBoard(row, col)) {
                // Calculate Manhattan distance to ensure the tile is within currentMoveRange
                let distance = Math.abs(row - rowIndex) + Math.abs(col - colIndex);
                if (distance >= minRange && distance <= maxRange) {
                    board[row][col].highlight = true;
                }
            }
        }
    }
}

let turnSystem = reactive([{ entity: { ...player }, onTurn: false }, { entity: { ...enemy }, onTurn: false }, { entity: { ...enemy }, onTurn: false }, { entity: { ...enemy }, onTurn: false }])

let board = reactive(Array.from({ length: numRows }, () =>
    Array.from({ length: numCols }, () => ({
        occupied: false,
        selected: false,
        icon: '',
        text: '',
        highlight: false,
    })))
);

function loadBoard() {
    board[0][0] = turnSystem[0].entity
    board[7][5] = turnSystem[1].entity
    board[7][6] = turnSystem[2].entity
    board[7][7] = turnSystem[3].entity
}

function rollLeft(arr) {
    if (arr.length === 0) return arr;
    arr.push(arr.shift());
    return arr;
}

function startTurn() {
    turnSystem[0].onTurn = true;
    turnSystem[0].entity.onTurn = true;
}

function stopTurn() {
    turnSystem[0].onTurn = false;
    turnSystem[0].entity.onTurn = false;
    turnSystem[0].entity.currentMoveRange = turnSystem[0].entity.maxMoveRange
    turnSystem[0].entity.currentActionPoints = turnSystem[0].entity.maxActionPoints
}

function skipOption() {
    skip();
}

function skip() {
    gameState.transition('skip');
    stopTurn()
    rollLeft(turnSystem)
    startTurn()
    skipTransition()
}

function moveOption() {
    if (turnSystem[0].entity.currentMoveRange == 0) {
        alert("No movement points left");
        return;
    }
    gameState.transition('moveOption');
    showAccessibleTiles(turnSystem[0].entity)
}

function move(obj, targetRow, targetCol) {
    gameState.transition('move')
    let startPosition = findObjectBoardPosition(obj);
    board[targetRow][targetCol] = board[startPosition.rowIndex][startPosition.colIndex]
    board[startPosition.rowIndex][startPosition.colIndex] = { ...empty }
    let distance = Math.abs(targetRow - startPosition.rowIndex) + Math.abs(targetCol - startPosition.colIndex);
    board[targetRow][targetCol].currentMoveRange -= distance;
    unhighlightBoard()
    gameState.transition('playerOptions')
}

function attackOption() {
    gameState.transition('attackOption')

}

function showAccessibleTiles(obj) {
    let position = findObjectBoardPosition(obj);
    if (!position) {
        console.error("Object not found on the board");
    }

    let rowIndex = position.rowIndex;
    let colIndex = position.colIndex;
    let currentMoveRange = obj.currentMoveRange;

    // Function to check if a position is within the board boundaries
    function isWithinBoard(row, col) {
        return row >= 0 && row < board.length && col >= 0 && col < board[0].length;
    }

    // Set tiles that are in range of obj.currentMoveRange to highlight
    for (let row = rowIndex - currentMoveRange; row <= rowIndex + currentMoveRange; row++) {
        for (let col = colIndex - currentMoveRange; col <= colIndex + currentMoveRange; col++) {
            if (isWithinBoard(row, col)) {
                // Calculate Manhattan distance to ensure the tile is within currentMoveRange
                let distance = Math.abs(row - rowIndex) + Math.abs(col - colIndex);
                if (distance == 0) continue;
                if (distance <= currentMoveRange) {
                    board[row][col].highlight = true;
                }
            }
        }
    }
    // set tiles that are in range of obj.moveRange to highlight
}

function findObjectBoardPosition(obj) {
    for (let row = 0; row < board.length; row++) {
        for (let col = 0; col < board[row].length; col++) {
            if (board[row][col] === obj) {
                return { rowIndex: row, colIndex: col };
            }
        }
    }
    return null; // Object not found on the board
}

function skipTransition() {
    if (turnSystem[0].entity.player) {
        gameState.transition('player');
        enterPlayerState();
    }
    else {
        gameState.transition('enemy')
    }
}

function enterPlayerState() {
    gameState.transition('playerOptions')

}

function initiateTurnsystem() {
    startTurn()
    console.log(board[0][0])
    skipTransition()
}

function startPress() {
    loadBoard()
    gameState.transition('skip')
    initiateTurnsystem()
}

function computeColor(rowIndex, colIndex, isHovering) {
    if (gameState.getCurrentState() == 'moveOption') {
        if (board[rowIndex][colIndex].highlight) {
            return "yellow"
        }
    }
    if (gameState.getCurrentState() == 'attackPlacement') {
        if (board[rowIndex][colIndex].highlight) {
            return "red"
        }
    }

    if (board[rowIndex][colIndex].selected) {
        return "yellow"
    }
    if (isHovering) {
        return "white";
    }
    else return ((rowIndex + colIndex) % 2) ? "primary" : "secondary"
}

function deselectBoard() {
    for (let i = 0; i < numRows; i++) {
        for (let j = 0; j < numRows; j++) {
            if (board[i][j].selected) {
                board[i][j].selected = false;
                return [i, j];
            }
        }
    }
    return false;
}

function unhighlightBoard() {
    for (let i = 0; i < numRows; i++) {
        for (let j = 0; j < numRows; j++) {
            board[i][j].highlight = false;
        }
    }
}

function getAffected(rowIndex, colIndex, attack){
    if (attack.AOE == 'single') {
        if (board[rowIndex][colIndex].isEntity){
            return [board[rowIndex][colIndex]]
        }
    } 

    return []
}

function applyAttack(affected, attack){
    if (affected.length == 0) return
    for (let i = 0; i < affected.length; i++) {
        affected[i].health -= attack.damage
    }
}

function executeAttack(rowIndex, colIndex, attack) {
    gameState.transition('attack')
    let affected = getAffected(rowIndex, colIndex, attack)
    turnSystem[0].entity.currentActionPoints -= attack.cost;
    applyAttack(affected, attack)

}

function cellClick(rowIndex, colIndex) {
    console.log("Here")
    if (gameState.getCurrentState() == 'moveOption') {
        if (!board[rowIndex][colIndex].highlight) {
            unhighlightBoard();
            gameState.transition('playerOptions')
            return
        }

        move(turnSystem[0].entity, rowIndex, colIndex)
    }

    if (gameState.getCurrentState() == 'attackOption') {
        gameState.transition('playerOptions')
    }

    if (gameState.getCurrentState() == 'attackPlacement') {
        if (!board[rowIndex][colIndex].highlight) {
            unhighlightBoard();
            gameState.transition('attackOption')
            return
        }
        executeAttack(rowIndex, colIndex, selectedAttack.value)
        unhighlightBoard();
        gameState.transition('attackOption')
    }

    if (gameState.getCurrentState() == 'idle') {
        return;
    }

}

class StateMachine {
    constructor(config) {
        this.states = config.states;
        this.transitions = config.transitions;
        this.currentState = config.initialState;
    }

    transition(event) {
        const stateTransitions = this.transitions[this.currentState];
        if (stateTransitions && stateTransitions[event]) {
            this.currentState = stateTransitions[event];
            console.log(this.currentState)
        } else {
            throw new Error(`Invalid transition from ${this.currentState} using ${event}`);
        }
    }

    canTransition(event) {
        const stateTransitions = this.transitions[this.currentState];
        return !!(stateTransitions && stateTransitions[event]);
    }

    getCurrentState() {
        return this.currentState;
    }
}

const gameStateConfig = {
    initialState: 'idle',
    states: ['idle', 'player', 'wait'],
    transitions: {
        'idle': { skip: 'skip' },
        'player': { playerOptions: 'playerOptions' },
        'playerOptions': { skip: 'skip', moveOption: 'moveOption', attackOption: 'attackOption' },
        'skip': { player: 'player', enemy: 'enemy' },
        'moveOption': { playerOptions: 'playerOptions', move: 'move' },
        'move': { playerOptions: 'playerOptions' },
        'attackOption': { attackPlacement: 'attackPlacement', playerOptions: 'playerOptions' },
        'attackPlacement': { attack: 'attack', attackOption: 'attackOption'},
        'attack': { attackOption: 'attackOption' },
        'enemy': { skip: 'skip' },
        'wait': { idle: 'idle', player: 'player', enemy: 'enemy', skip: 'skip' }
    }
};

const gameState = reactive(new StateMachine(gameStateConfig));

watch(show, (oldVal) => {
    show.value = true;
})

</script>

<style>
.grid-cell {
    aspect-ratio: 1;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    z-index: 100 !important;
}

.vertical-center {
    margin: 0;
    position: absolute;
    top: 50%;
    -ms-transform: translateY(-50%);
    transform: translateY(-50%);
}

.right-from-parent {
    margin: 0;
    position: absolute;
    top: 50%;
    -ms-transform: translateY(-50%);
    transform: translateY(-50%);
    margin-left: 100%;
    z-index: 10000000 !important;
}

.v-tooltip>.v-overlay__content {
    background: rgba(0, 0, 0, 0) !important;
}
</style>
