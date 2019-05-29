<template>
  <div id="game"></div>
</template>
<script>
/* eslint-disable */
import Phaser from "phaser";
import wall from "../assets/wall.png";
import ground from "../assets/ground.png";
import player from "../assets/player.png";
import coin from "../assets/coin.png";
import enemy from "../assets/enemy.png";
import die_sound from "../assets/dead.mp3";
import takecoin_sound from "../assets/coin.mp3";
import explosion from "../assets/explosion.png";
let score = 0;
let scoreText;
let lives = 3;
let livesText;

function die(player, enemy) {
  this.explosion = this.physics.add.sprite(
    player.body.x + player.body.height / 2,
    player.body.y + player.body.width / 2,
    "explosion"
  );
  this.explosion.anims.play("explosion", false);
  player.scene.cameras.main.shake(500)
  player.disableBody(true, true);
  setTimeout(() => {
    this.sound.play("die");
    this.cameras.main.shake(500);
    this.scene.restart();
    lives = lives - 1;
    livesText.setText("Lives: " + lives);
    if (lives == 0) {
      this.scene.stop();
    }
  }, 50);
}

function onWorldBoundsCollide() {
  this.explosion = this.physics.add.sprite(
    this.player.body.x + this.player.body.height / 2,
    this.player.body.y + this.player.body.width / 2,
    "explosion"
  );
  this.explosion.anims.play("explosion", false);
  this.player.disableBody(true, true);
  setTimeout(() => {
    this.sound.play("die");
    this.cameras.main.shake(500);
    this.scene.restart();
    lives = lives - 1;
    livesText.setText("Lives: " + lives);
    if (lives == 0) {
      this.scene.stop();
    }
  }, 50);
}

function takeCoin(player, coin) {
  coin.body.enable = false
  this.tweens.add({
    targets: coin,
    props: {
      y: { value: -800, duration: 500, ease: "Linear" }
    }
  });
  setTimeout(() => {
    coin.disableBody(true, true);
  }, 2000);
  this.sound.play("take_coin");
  score = score + 10;
  scoreText.setText("Score: " + score);
}
export default {
  name: "Game",
  mounted() {
    // PHASER 2.6 -> phaser-ce
    // PHASER 3.0 -> phaser
    let config = {
      type: Phaser.AUTO,
      width: 500,
      height: 200,
      physics: {
        default: "arcade",
        arcade: {
          gravity: { y: 200 }
        }
      },
      // NO HI HA STATES A 3.0 -> SCENES
      scene: {
        preload() {
          this.load.spritesheet("explosion", explosion, {
            frameWidth: 48,
            frameHeight: 48
          });
          // CARREGAR ASSETS -> IMAGES, PLAYERS (SPRITESHEETS), AUDIOS, VIDEOS
          this.load.image("wall", wall);
          this.load.image("ground", ground);
          this.load.spritesheet("player", player, {
            frameWidth: 28,
            frameHeight: 22
          });
          this.load.image("coin", coin);
          this.load.image("enemy", enemy);
          // AUDIO
          this.load.audio("die", die_sound);
          this.load.audio("take_coin", takecoin_sound);
        },
        create() {
          // INITILIZE del nivell -> Afegirem tiles (level: pareds, terras), afegirem player/s, collectibles (coins) , enemies
          this.sound.add("die");
          this.sound.add("take_coin");
          this.cameras.main.backgroundColor.setTo(52, 152, 219);
          // GROUP PER DEFECTE ES DINAMIC
          this.level = this.physics.add.staticGroup();
          this.level.create(500 / 2 - 160, 200 / 2, "wall");
          this.level.create(500 / 2 + 160, 200 / 2, "wall");
          this.level.create(500 / 2, 200 / 2 + 30, "ground");
          // PLAYER -> FÍSIQUES
          this.player = this.physics.add.sprite(
            500 / 2,
            200 / 2 - 50,
            "player"
          );
          this.player.setBounce(0.2);
          this.player.body.setCollideWorldBounds(true);
          this.player.body.onWorldBounds = true;
          this.physics.add.collider(this.player, this.level);
          // cursors
          this.cursors = this.input.keyboard.createCursorKeys();
          // ANIMATE PLAYER
          this.anims.create({
            key: "idle",
            frames: this.anims.generateFrameNumbers("player", {
              start: 3,
              end: 5
            }),
            frameRate: 5,
            repeat: -1
          });
          this.anims.create({
            key: "explosion",
            frames: this.anims.generateFrameNumbers("explosion", {
              start: 0,
              end: 6
            }),
            frameRate: 60,
            repeat: -1
          });
          // ADD COINS
          this.coins = this.physics.add.group();
          this.coins.create(140, 200 / 2, "coin");
          this.coins.create(170, 200 / 2, "coin");
          this.coins.create(200, 200 / 2, "coin");
          this.physics.add.collider(this.coins, this.level);
          this.physics.add.overlap(
            this.player,
            this.coins,
            takeCoin,
            null,
            this
          );
          // SCORE
          scoreText = this.add.text(10, 10, "Score: " + score, {
            fontSize: "12px",
            fill: "#000"
          });
          livesText = this.add.text(10, 30, "Lives: " + lives, {
            fontSize: "12px",
            fill: "#000"
          });
          // ENEMIES
          this.enemies = this.physics.add.group();
          this.enemies.create(500 / 2 + 130, 200 / 2, "enemy");
          this.physics.add.collider(this.enemies, this.level);
          this.physics.add.overlap(this.player, this.enemies, die, null, this);
          this.physics.world.on("worldbounds", onWorldBoundsCollide, this);
          //  prepare loser txt
          this.loserText = this.add.text(500 / 2, 200 / 2 - 50, 'You lose!', { fontSize: '24px', fill: '#000' }).setVisible(false)
          this.cameras.main.on('camerashakestart', () => {
            this.loserText.setVisible(true)
          })
          this.cameras.main.on('camerashakecomplete', () => {
            this.loserText.setVisible(false)
          })
        },
        update() {
          this.player.anims.play("idle", true);
          // INPUT EVENTS
          if (this.cursors.left.isDown) {
            this.player.setVelocityX(-160);
            this.player.setFrame(2);
          } else if (this.cursors.right.isDown) {
            this.player.setVelocityX(160);
            this.player.setFrame(1);
          } else {
            this.player.setVelocityX(0);
          }
          // JUMP
          if (this.cursors.up.isDown && this.player.body.touching.down) {
            this.player.setVelocityY(-160);
          }
          // console.log('UPDATED')
          // ESTE EL QUE S'executa continuament al Game loop -> 60 vegades per segon o FPS
          // if (this.player.body.touching.down && this.player.y > 10) {
          //   this.dustSound.play()
          // }
        }
      }
    };
    // eslint-disable-next-line
    new Phaser.Game(config);
  }
};

</script>
