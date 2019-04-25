<template>
    <div id="game"></div>
</template>

<script>
import Phaser from 'phaser'
import wall from '../assets/wall.png'
import ground from '../assets/ground.png'
import player from '../assets/player.png'
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
          this.player.setCollideWorldBounds(true)
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
