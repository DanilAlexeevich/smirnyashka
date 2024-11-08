# smirnyashka

``` javascript
update(keys) {
        const INACTIVITY_THRESHOLD = 1000;
        const INACTIVITY_Breeze = 200;
        // Принудительно бьемся головой об потолок
        if (this.sprite.y < 50) { // Высота header
            this.sprite.y = 50; // Принудительно устанавливаем y на 50
            this.sprite.setVelocityY(0); // Останавливаем вертикальное движение
        }

        if (keys.keyD.isDown) {
            this.inactivityTimer = 0;
            this.sprite.setVelocityX(this.speed);
            this.lookRight = true;
            if (this.sprite.body.touching.down && (!this.isJumping || this.jumpIsOver)) {
                if (!this.withBlade) {
                    this.sprite.anims.play('runRight', true);
                } else {
                    this.sprite.anims.play('runWR', true);
                }
                
            }   
        } else if (keys.keyA.isDown) {
            this.inactivityTimer = 0;
            this.sprite.setVelocityX(-this.speed);
            this.lookRight = false;
            if (this.sprite.body.touching.down && (!this.isJumping || this.jumpIsOver)) {
                if (!this.withBlade) {
                    this.sprite.anims.play('runLeft', true);
                } else {
                    this.sprite.anims.play('runWL', true);
                }
            }
        } else if ( this.sprite.body.touching.down && !this.isJumping) {
            this.sprite.setVelocityX(0);

            // Если текущая анимация прыжка, остановим её и вернем на начальный кадр
            if (this.sprite.anims.currentAnim && 
                (this.sprite.anims.currentAnim.key === 'playerJumpR' || this.sprite.anims.currentAnim.key === 'playerJumpL')) {
                
                this.sprite.anims.stop();
                this.sprite.setFrame(0); // Устанавливаем дефолтный кадр
        
            } else {
                this.inactivityTimer += this.delta;
                // Если персонаж с мечом, проигрываем стояние с мечом
                if (this.withBlade) {
                    if (this.lookRight) {
                        this.sprite.anims.play('standWR', true);
                    } else {
                        this.sprite.anims.play('standWL', true);
                    }
                } else if (this.lookRight && !this.isAttacking ){
                    // Иначе обычная анимация стоя
                    if(this.inactivityTimer >= INACTIVITY_THRESHOLD){
                        this.sprite.anims.play('standHelloR', true);
                    } else this.sprite.anims.play('standR', true);
                } else if (!this.isAttacking) {
                    if(this.inactivityTimer >= INACTIVITY_THRESHOLD){
                        this.sprite.anims.play('standHelloL', true);
                    } else this.sprite.anims.play('standL', true);
                }       
            }
        }
        


        // Проверка нажатия пробела для прыжка
        if (keys.keySpace.isDown && this.canJump && this.jumpCount < this.maxJumps) {
            this.sprite.setVelocityY(this.jumpPower);
            
            if (this.jumpCount === 1) {
                if (this.lookRight) {
                    if (!this.withBlade) {
                        this.sprite.play('playerJumpR', true); 
                    } else {
                        this.sprite.play('playerJumpWithR', true); 
                    }
                } else if (!this.withBlade) {
                    this.sprite.play('playerJumpL', true);
                } else {
                    this.sprite.play('playerJumpWithL', true); 
                }
                this.jumpIsOver = true; // Воспроизведение анимации только при втором прыжке
            }
            this.inactivityTimer = 0;
            this.isJumping = true;
            console.log('аним');
            this.jumpCount++;
            this.canJump = false;
            this.jumpIsOver = false;
             // Запретить дальнейшие прыжки до отпускания
        } 
        if (keys.keyF.isDown) {
            if(this.lookRight){
                this.sprite.x += 40;
            } else this.sprite.x -= 40;
        }
        // Проверка отпускания клавиши пробела
        if (keys.keySpace.isUp) {
            this.canJump = true; // Позволить прыгать снова после отпускания
        }

        // Сброс счетчика прыжков при касании земли
        if (this.sprite.body.touching.down) {
            this.jumpCount = 0; // Сброс счетчика прыжков
            this.canJump = true; // Сбросить флаг для разрешения прыжков
            this.isJumping = false; 
        }
        //АТАКА

       // Проверка завершения анимации атаки и сброс флага
        this.sprite.on('animationcomplete', (animation) => {
         if (animation.key.startsWith('attack')) {
            this.isAttacking = false;
            console.log('Атака завершена'); // Отладка
        }
        });

        this.sprite.on('animationcomplete', (animation) => {
            if (animation.key === 'standHelloL' || animation.key === 'standHelloR') {
                // Сброс таймера после завершения анимации
                this.inactivityTimer = 0;
            }
        });
    }



performAttack() {
        // Увеличиваем шаг комбо и проверяем, на каком этапе анимация
        this.attackStep++;
        if(this.lookRight){
            if (this.attackStep === 1) {
                this.sprite.play('attack1R');
            } else if (this.attackStep === 2) {
                this.sprite.play('attack2R');
            } else if (this.attackStep === 3) {
                this.sprite.play('attack3R');
                this.attackStep = 0; // Сброс после последнего удара
            }
        }else {
            if (this.attackStep === 1) {
                this.sprite.play('attack1L');
            } else if (this.attackStep === 2) {
                this.sprite.play('attack2L');
            } else if (this.attackStep === 3) {
                this.sprite.play('attack3L');
                this.attackStep = 0; // Сброс после последнего удара
            }
        }
        // Перезапуск таймера для сброса комбо
        clearTimeout(this.attackTimeout);
        this.attackTimeout = setTimeout(() => {
            this.attackStep = 0;
        }, 1000); // Тайм-аут для сброса комбинации (500 мс)
    }






дыолвгшаищуцршащицушсрищци


import PlayerController from './controller.js';
import { monsters } from './levels/level1.js';


export class Player {
    // Общие свойства персонажа
    constructor(scene, x, y) {
        this.scene = scene;
        this.sprite = scene.physics.add.sprite(x, y, 'playerRun');
        // Размер спрайта
        this.sprite.setScale(3);
        // Теперь настраиваем коллайдер с учетом масштаба
        this.sprite.body.setSize(48 / 3.5, 48 / 2);
        this.sprite.body.setOffset(17, 15);
        this.sprite.setCollideWorldBounds(true);
        // Характеристики и состояния 
        this.speed = 150;
        this.jumpPower = -270;
    
        this.jumpCount = 0;
        this.maxJumps = 2;

        this.died = false;


        this.isJumping = false;

        this.isFalling = false;

        this.lookRight = true;

        this.withBlade = false;

        this.inactivityTimer = 0;
        this.delta = 1;
        this.attackStep = 0; // Текущий шаг комбо
        this.attackTimeout; // Таймер для сброса комбо
        this.isAttacking = false; 
        this.attackQueue = []; // Флаг для анимации атаки

        const keys = scene.input.keyboard.addKeys({
            keyA: Phaser.Input.Keyboard.KeyCodes.A,
            keyD: Phaser.Input.Keyboard.KeyCodes.D,
            keySpace: Phaser.Input.Keyboard.KeyCodes.SPACE,
            keyF: Phaser.Input.Keyboard.KeyCodes.F,
            attackButton: Phaser.Input.Keyboard.KeyCodes.C
        });
        this.createAnimations();
        this.controller = new PlayerController(this, keys, scene);
        this.sprite.body.onWorldBounds = true;
        this.scene.physics.world.on('worldbounds', this.onWorldBounds, this);
    }

    createAnimations(){
        this.scene.anims.create({
            key: 'runRight',
            frames: this.scene.anims.generateFrameNumbers('playerRun', { start: 0, end: 5 }),
            frameRate: 10,
            repeat: -1
        });

        this.scene.anims.create({
            key: 'runLeft',
            frames: this.scene.anims.generateFrameNumbers('playerRun', { start: 6, end: 11 }),
            frameRate: 10,
            repeat: -1
        });

        this.scene.anims.create({
            key: 'runWRight',
            frames: this.scene.anims.generateFrameNumbers('runWithBlade', { start: 0, end: 5 }),
            frameRate: 10,
            repeat: -1
        });
        
        this.scene.anims.create({
            key: 'runWLeft',
            frames: this.scene.anims.generateFrameNumbers('runWithBlade', { start: 6, end: 11 }),
            frameRate: 10,
            repeat: -1
        });

        this.scene.anims.create({
            key: 'playerJumpR',
            frames: this.scene.anims.generateFrameNumbers('playerJumpR', { start: 0, end: 8 }),
            frameRate: 13,
            repeat: 0
        });

        this.scene.anims.create({
            key: 'playerJumpL',
            frames: this.scene.anims.generateFrameNumbers('playerJumpL', { start: 0, end: 8 }),
            frameRate: 13,
            repeat: 0
        });

        this.scene.anims.create({
            key: 'playerJumpWithR',
            frames: this.scene.anims.generateFrameNumbers('playerJumpWithBlade', { start: 0, end: 8 }),
            frameRate: 13,
            repeat: 0
        });

        this.scene.anims.create({
            key: 'playerJumpWithL',
            frames: this.scene.anims.generateFrameNumbers('playerJumpWithBlade', { start: 9, end: 17 }),
            frameRate: 13,
            repeat: 0
        });
        this.scene.anims.create({
            key: 'standR',
            frames: this.scene.anims.generateFrameNumbers('stand', { start: 0, end: 7 }),
            frameRate: 3,
            repeat: -1
        });
        this.scene.anims.create({
            key: 'standL',
            frames: this.scene.anims.generateFrameNumbers('stand', { start: 8, end: 15 }),
            frameRate: 3,
            repeat: -1
        });
        this.scene.anims.create({
            key: 'standWR',
            frames: this.scene.anims.generateFrameNumbers('standWithBlade', { start: 0, end: 5 }),
            frameRate: 10,
            repeat: -1
        });
        this.scene.anims.create({
            key: 'standWL',
            frames: this.scene.anims.generateFrameNumbers('standWithBlade', { start: 6, end: 11 }),
            frameRate: 10,
            repeat: -1
        });
        this.scene.anims.create({
            key: 'standHelloR',
            frames: this.scene.anims.generateFrameNumbers('standHello', { start: 0, end: 14 }),
            frameRate: 10,
            repeat: 0
        });
        this.scene.anims.create({
            key: 'standHelloL',
            frames: this.scene.anims.generateFrameNumbers('standHello', { start: 15, end: 29 }),
            frameRate: 10,
            repeat: 0
        });
        //PUNCH COMBO R
        this.scene.anims.create({
            key: 'attack1R',
            frames: this.scene.anims.generateFrameNumbers('attack', { start: 0, end: 3 }),
            frameRate: 10,
            repeat: 0
        });
        this.scene.anims.create({
            key: 'attack2R',
            frames: this.scene.anims.generateFrameNumbers('attack', { start: 4, end: 6 }),
            frameRate: 10,
            repeat: 0
        });
        this.scene.anims.create({
            key: 'attack3R',
            frames: this.scene.anims.generateFrameNumbers('attack', { start: 7, end: 10 }),
            frameRate: 10,
            repeat: 0
        });
        //PUNCH COMBO L
        this.scene.anims.create({
            key: 'attack1L',
            frames: this.scene.anims.generateFrameNumbers('attack', { start: 12, end: 15 }),
            frameRate: 10,
            repeat: 0
        });
        this.scene.anims.create({
            key: 'attack2L',
            frames: this.scene.anims.generateFrameNumbers('attack', { start: 16, end: 18 }),
            frameRate: 10,
            repeat: 0
        });
        this.scene.anims.create({
            key: 'attack3L',
            frames: this.scene.anims.generateFrameNumbers('attack', { start: 19, end: 23 }),
            frameRate: 10,
            repeat: 0
        });

    }
   
    onWorldBounds(body) {
        if (body.gameObject === this.sprite) {
            this.died = true;
        }
    }

    pickupSword(){
        this.withBlade = true;
        this.sprite.setTexture('standWR');
    }
    timerAn() {
        // Используем this.scene.time вместо this.time
        this.scene.time.delayedCall(5000, () => {
            this.tenSec = true; // Например, вызов атаки персонажа
        });
    }

    
    playRunAnimation(direction) {
        const runAnim = this.withBlade ? `runW${direction}` : `run${direction}`;
        this.sprite.anims.play(runAnim, true);
    }

    playIdleAnimation(inactivityTimer, threshold) {
        const idleAnim = this.withBlade ? `standW${this.lookRight ? 'R' : 'L'}` : `stand${this.lookRight ? 'R' : 'L'}`;
        const helloAnim = `standHello${this.lookRight ? 'R' : 'L'}`;

        if (inactivityTimer >= threshold) {
            this.sprite.anims.play(helloAnim, true);
        } else {
            this.sprite.anims.play(idleAnim, true);
        }
    }

    playJumpAnimation(lookRight, withBlade, jumpCount) {
        if (jumpCount === 1) {
            const jumpAnim = withBlade
                ? `playerJumpWith${lookRight ? 'R' : 'L'}`
                : `playerJump${lookRight ? 'R' : 'L'}`;
            this.sprite.play(jumpAnim, true);
            this.jumpIsOver = true;
        }
    }

    resetJump() {
        this.jumpCount = 0;
        this.canJump = true;
        this.isJumping = false;
        this.isAttackBlocked = false;
    }

    // Че делает
    update(keys) {
        this.controller.update();
    }
}
export default Player;




// playerController.js
class PlayerController {
    constructor(player, keys, scene) {
        this.player = player;
        this.sprite = player.sprite;
        this.keys = keys;
        this.inactivityTimer = 0;
        this.delta = 1; // Задай значение delta, если оно приходит извне
        this.INACTIVITY_THRESHOLD = 1000;
        this.INACTIVITY_BREEZE = 200;
        this.lookRight = true;
        // this.isAttacking = false;
        this.comboStage = 0; // Этап комбо
        this.comboTimeout = 500; // Таймаут для сброса комбо (мс)
        this.comboTimer = null;
        this.withBlade = false;

        scene.input.on('pointerdown', (pointer) => {
            if (pointer.leftButtonDown()) {
                this.handleAttack();
                console.log("тычкатычка");
            }
        });

        
    }

    handleCeilingCollision() {
        if (this.player.sprite.y < 50) { 
            this.player.sprite.y = 50;
            this.player.sprite.setVelocityY(0);
        }
    }

    handleMovement() {
        if (this.isAttacking) {
            this.player.sprite.setVelocityX(0);
            return;
        }
        if (this.keys.keyD.isDown) {
            this.inactivityTimer = 0;
            this.player.sprite.setVelocityX(this.player.speed);
            this.player.lookRight = true;

            if (this.player.sprite.body.touching.down && (!this.player.isJumping || this.player.jumpIsOver)) {
                this.player.playRunAnimation('Right');
            }
        } else if (this.keys.keyA.isDown) {

            this.inactivityTimer = 0;
            this.player.sprite.setVelocityX(-this.player.speed);
            this.player.lookRight = false;

            if (!this.withBlade && this.player.sprite.body.touching.down && (!this.player.isJumping || this.player.jumpIsOver)) {
                this.player.playRunAnimation('Left');
            }
            
        } else {
            this.player.sprite.setVelocityX(0);
            this.handleInactivity();
        }
    }

    handleInactivity() {
        if (this.isAttacking) return;
        this.inactivityTimer += this.delta;

        if (this.player.sprite.body.touching.down && !this.player.isJumping) {
            this.player.playIdleAnimation(this.inactivityTimer, this.INACTIVITY_THRESHOLD);
        }
    }

    handleJump() {
        if (this.keys.keySpace.isDown && this.player.canJump && this.player.jumpCount < this.player.maxJumps) {

            this.player.sprite.setVelocityY(this.player.jumpPower);
            // Если это второй прыжок (сальто), блокируем атаки
            if (this.player.jumpCount === 1) {
                this.player.isAttackBlocked = true;
            }
            
            this.player.playJumpAnimation(this.player.lookRight, this.player.withBlade, this.player.jumpCount);

            this.inactivityTimer = 0;
            this.player.isJumping = true;
            this.player.jumpCount++;
            
            this.player.canJump = false;
            this.player.jumpIsOver = false;
        }

        if (this.keys.keySpace.isUp) {
            this.player.canJump = true;
        }

        if (this.player.sprite.body.touching.down) {
            this.player.resetJump();
            this.player.isAttackBlocked = false;
        }
    }

    handleAttack() {
        // If already attacking, queue next combo stage (up to 3 stages)
        if (this.isAttacking) {
            if (this.comboQueue.length < 3) {
                this.comboQueue.push(this.comboQueue.length + 1);
            }
        } else {
            // Start the combo from the first attack
            this.isAttacking = true;
            this.comboQueue = [1];
            this.processCombo();
        }
        // Reset combo timer to prevent combo reset during fast clicks
        clearTimeout(this.comboTimer);
        this.comboTimer = setTimeout(() => {
            this.isAttacking = false;
            this.comboQueue = []; // Clear queue after delay
        }, this.comboTimeout);
    }

    processCombo() {
        if (this.comboQueue.length === 0) {
            this.isAttacking = false;
            this.comboStep = 1; // Reset combo step
            return;
        }
    
        // Get the current attack stage from the queue
        this.comboStep = this.comboQueue.shift();
    
        // Determine animation key based on direction and weapon presence
        const attackPrefix = this.withBlade ? 'bladeAttack' : 'attack';
        const direction = this.lookRight ? 'R' : 'L';
        const animationKey = `${attackPrefix}${this.comboStep}${direction}`;
    
        console.log(`Playing animation: ${animationKey}`);
        this.sprite.anims.play(animationKey, true);
    
        // After animation completes, check for next combo stage
        this.sprite.once('animationcomplete', (animation) => {
            if (animation.key.startsWith(attackPrefix)) {
                console.log("Attack animation complete");
                this.processCombo(); // Trigger the next attack stage in queue
            }
        });
    }

    handleAnimationEvents() {
        this.player.sprite.on('animationcomplete', (animation) => {
            if (animation.key.startsWith('attack')) {
                this.player.isAttacking = false;
            }
            if (animation.key === 'standHelloL' || animation.key === 'standHelloR') {
                this.inactivityTimer = 0;
            }
        });
    }

    update() {
        this.handleCeilingCollision();
        this.handleMovement();
        this.handleJump();
        this.handleAnimationEvents();
        // if (this.keys.attackButton.isDown) {
        //     this.handleAttack();
        //     console.log('тычка');
        // }

        // Проверка направления персонажа
        if (this.keys.keyD.isDown) {
            this.lookRight = true;
        } else if (this.keys.keyA.isDown) {
            this.lookRight = false;
        }
    }
}

export default PlayerController;




import { Player } from './player.js';
import { createLevel1, levelConfig, resetLevelObjects } from './levels/level1.js';
import { createLevel2, levelConfig2, resetLevelObjects2 } from './levels/level2.js';
// import { PlayerController } from './controller.js'

let player, controller;
let game;
let currentLevel = 1;
let targetOffsetX = -200;

const config = {
    type: Phaser.AUTO,
    width: 1400,
    height: 2048,
    parent: 'gameContainer',
    physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 500 },
            debug: false
        }
    },
    scene: {
        preload: preload,
        create: create,
        update: update
    },
    render: {
        pixelArt: true
    }
};

function preload() {
    this.load.spritesheet('playerRun', 'resources/animations/playerRunNS.png', {
        frameWidth: 48,
        frameHeight: 48
    });
    this.load.spritesheet('platformSprite', 'resources/animations/plats.png', {
        frameWidth: 32,
        frameHeight: 32,        
    });
    this.load.spritesheet('playerJumpR', 'resources/animations/playerJumpR.png', {
        frameWidth: 48,
        frameHeight: 48
    });
    this.load.spritesheet('slimeSprite', 'resources/animations/slimeSprite.png',{
        frameWidth: 48, 
        frameHeight: 48 
    });
    this.load.spritesheet('playerJumpL', 'resources/animations/playerJumpL.png', {
        frameWidth: 48,
        frameHeight: 48
    });
    this.load.spritesheet('playerJumpWithBlade', 'resources/animations/playerJumpWithBlade.png', {
        frameWidth: 48,
        frameHeight: 48
    });
    
    this.load.spritesheet('stand', 'resources/animations/stand.png', {
        frameWidth: 48,
        frameHeight: 48
    });
    this.load.spritesheet('standWithBlade', 'resources/animations/playerStayWithBlade.png', {
        frameWidth: 48,
        frameHeight: 48
    });
    this.load.spritesheet('runWithBlade', 'resources/animations/playerRunBlade.png', {
        frameWidth: 48,
        frameHeight: 48
    });
    this.load.spritesheet('swordWaiter', 'resources/animations/bladeWaiter.png', {
        frameWidth: 48,
        frameHeight: 48
    });
    this.load.spritesheet('standHello', 'resources/animations/standHello.png', {
        frameWidth: 48,
        frameHeight: 48
    });
    this.load.spritesheet('attack', 'resources/animations/PunchCombo.png', {
        frameWidth: 48,
        frameHeight: 48
    });
}       

export function setupCamera(scene) {
    scene.cameras.main.startFollow(player.sprite, true, 0.1, 0.1, 0, 200); // Следит за игроком с мягкой плавностью
    scene.cameras.main.setBounds(0, 0, config.width, config.height);
    scene.cameras.main.setSize(config.width, 1000);
    scene.cameras.main.setViewport(0, 100, 1200, 800);
    scene.cameras.main.setFollowOffset(targetOffsetX, 0);
    scene.cameras.main.setDeadzone(0, 20);
    scene.cameras.main.setZoom(1.3);
    scene.cameras.main.setBackgroundColor('#ffb7c5');
}

export function zoomInOnObject(scene, zoomLevel = 2, duration = 1000) {
    const target = scene.swordWaiter;  // Используем `sword` из `scene`
    if (target && target.x !== undefined && target.y !== undefined) {
        scene.cameras.main.pan(target.x, target.y, duration, 'Power2');
        scene.cameras.main.zoomTo(zoomLevel, duration);
    } else {
        console.error('Target for zoomInOnObject is not defined or has invalid coordinates');
    }
    setupCamera(scene);
}

//******************СОЗДАЕМ****************************
function create() {
    // Создаем клавиши управления
    this.keyA = this.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.A);
    this.keyD = this.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.D);
    this.keySpace = this.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.SPACE);
    this.keyF = this.input.keyboard.addKey(Phaser.Input.Keyboard.KeyCodes.F);
    this.mouseClick = this.input.activePointer.isDown;
    

    // Создаём player и инициализируем sprite
    player = new Player(this, levelConfig.playerStart.x, levelConfig.playerStart.y);

    // Теперь создаем controller, передавая player, keys и scene

    // Привязываем controller к player для использования в других частях

    // Я всегда с собой беру, видеокамеру
    // setupCamera(this);
    // Создаем уровень
    // createLevel.call(this, currentLevel, player);
    resetLevel(this); // передача контекста this
    setupCamera(this);
}

// Метод для сброса уровня
function resetLevel(scene) {
    // Удаляем все объекты уровня
    scene.children.removeAll();
    player.died = false;
    console.log('You died');
    
    // Создаём игрока заново на начальной позиции
    player = new Player(scene, levelConfig.playerStart.x, levelConfig.playerStart.y, controller);
    
    setupCamera(scene);

    // Создаём уровень, соответствующий currentLevel
    createLevel.call(scene, currentLevel, player);

}

// Функция для создания уровня в зависимости от currentLevel
function createLevel(level, player) {
    const scene = this;
    // console.log(this);
    // Создание уровня в зависимости от currentLevel
    if (level === 1) {
        createLevel1(this, player);
    } else if (level === 2) {
        createLevel2(this, player);
    } 
    // Добавьте дополнительные условия для других уровней
}

//********************АПДЕЙТ***************************
function update() {
    if (player.died) {
        resetLevel(this); // Если персонаж мертв, перезагрузить уровень
    } else {
        player.update();
        //{ keyA: this.keyA, keyD: this.keyD, keySpace: this.keySpace, keyF: this.keyF, mouseClick: this.mouseClick }
        if (this.keyD.isDown) {
            targetOffsetX = -250; // Смещение влево
        } else if (this.keyA.isDown) {
            targetOffsetX = 250; // Смещение вправо
        }
        const currentOffsetX = this.cameras.main.followOffset.x;
        this.cameras.main.setFollowOffset(
            Phaser.Math.Interpolation.Linear([currentOffsetX, targetOffsetX], 0.01), 
            0 // Вертикальное смещение остаётся на 0
        );
    } 
}    

// Создание игры
game = new Phaser.Game(config);
```
