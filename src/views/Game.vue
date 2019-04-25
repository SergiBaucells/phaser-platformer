<template>
    <div id="game"></div>
</template>

<script>
import Phaser from 'phaser'
import wall from '../assets/wall.png'
import ground from '../assets/ground.png'
import player from '../assets/player.png'
import coin from '../assets/coin.png'
import enemy from '../assets/enemy.png'

let score = 0
let scoreText

// TODO: Detectar quan l'usuari surt del mon i executar un die

function takeCoin (player, coin) {
  // TODO: Executar el so que toca (coin.mp3) + Actualitzar un score de punts
  coin.disableBody(true, true)
  score = score + 10
  // this.add.text(10, 10, 'Score: ' + score, { fontSize: '12px', fill: '#000' })
  scoreText.setText('Score: ' + score)
}

function die (player, enemy) {
  // TODO: Executar el so que toca (coin.mp3)
  // TODO: Actualitzar un score de punts
  // TODO: Shake
  // TODO: Reiniciar lvl
  // TODO: Display emb el número de vides, mostrar-les i si s'agotes s'atura el joc
  // TODO: Animacions
  player.disableBody(true, true)
}

export default {
  name: 'Game',
  mounted () {
    // Phase 3.0 -> phaser
    let config = {
      type: Phaser.AUTO,
      width: 500,
      height: 200,
      physics: {
        default: 'arcade',
        arcade: {
          gravity: { y: 200 }
        }
      },
      // No hi ha states a 3.0, hi ha scenes (pantalles)
      scene: {
        preload () {
          // Carregar assets -> imatges, players, audios, videos etc.

          // Imatges
          this.load.image('wall', wall)
          this.load.image('ground', ground)
          this.load.image('coin', coin)
          this.load.image('enemy', enemy)
          this.load.spritesheet('player', player, { frameWidth: 28, frameHeight: 22 })

          // Audio
          // this.load.setBaseUrl('http://labs.phaser.io')
          // TODO: Mirar si funciona el audio
          // this.load.audio('dust', ['../assets/dust.wav', '../assets/dust.mp3'])
        },
        create () {
          // Inicialitzador del nivell

          // Fons
          this.cameras.main.backgroundColor.setTo(52, 152, 219)

          // Grup per defecte es dinàmic
          // this.level = this.physics.add.group()
          this.level = this.physics.add.staticGroup()

          // Mon
          // this.add.image(500 / 2 - 160, 200 / 2, 'wall')
          // this.add.image(500 / 2 + 160, 200 / 2, 'wall')
          // this.ground = this.physics.add.image(500 / 2, 200 / 2 + 30, 'ground')
          this.level.create(500 / 2 - 160, 200 / 2, 'wall')
          this.level.create(500 / 2 + 160, 200 / 2, 'wall')
          this.level.create(500 / 2, 200 / 2 + 30, 'ground')

          // this.ground.body.allowGravity = false

          // Player
          this.player = this.physics.add.sprite(500 / 2, 200 / 2 - 50, 'player')
          // this.player.setCollideWorldBounds(true)
          this.player.setBounce(0.2)
          this.physics.add.collider(this.player, this.level)

          // Define sounds
          // this.dustSound = this.add.audio('dust', 0.1)

          // Cursors
          this.cursors = this.input.keyboard.createCursorKeys()

          // Animate player
          this.anims.create({
            key: 'idle',
            frames: this.anims.generateFrameNumbers('player', { start: 3, end: 5 }),
            frameRate: 5,
            repeat: -1
          })

          // Add coins
          this.coins = this.physics.add.group()
          this.coins.create(140, 200 / 2, 'coin')
          this.coins.create(170, 200 / 2, 'coin')
          this.coins.create(200, 200 / 2, 'coin')
          this.physics.add.collider(this.coins, this.level)
          this.physics.add.overlap(this.player, this.coins, takeCoin, null, this)

          // Score
          scoreText = this.add.text(10, 10, 'Score: 0', { fontSize: '12px', fill: '#000' })

          // Enemy
          this.enemies = this.physics.add.group()
          this.enemies.create(500 / 2 + 130, 200 / 2, 'enemy')
          this.physics.add.collider(this.enemies, this.level)
          this.physics.add.overlap(this.player, this.enemies, die, null, this)
        },
        update () {
          this.player.anims.play('idle', true)
          // S'executa continuament al gameloop -> 60 vegades per segon o FPS

          // if (this.player.body.touching.down && this.player.y > 10) {
          //   this.dustSound.play()
          // }

          // Input Events
          if (this.cursors.left.isDown) {
            this.player.setVelocityX(-160)
            this.player.setFrame(1)
          } else if (this.cursors.right.isDown) {
            this.player.setVelocityX(160)
            this.player.setFrame(2)
          } else {
            this.player.setVelocityX(0)
          }

          // Jump
          if (this.cursors.up.isDown && this.player.body.touching.down) {
            this.player.setVelocityY(-160)
          }
        }
      }
    }
    // eslint-disable-next-line
      new Phaser.Game(config)
  }
}
</script>
