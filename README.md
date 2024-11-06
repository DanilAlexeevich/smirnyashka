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
```
