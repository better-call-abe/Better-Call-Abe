<template>
  <div class="mt-12 max-w-6xl mx-auto flex justify-center">
    <div>
      <div>
        <h3 class="text-3xl font-semibold"></h3>
      </div>
      <div
        class="grid grid-cols-1"
        :class="[localJoined ? 'grid-cols-2 gap-4' : '']"
      >
        <div ref="videoChatWindow"></div>
        <!-- <div ref="remoteChatWindown"></div> -->
      </div>
      <button
        v-if="!localJoined"
        @click.prevent="joinRoom"
        class="mt-4 bg-red-600 py-3 px-6 text-red-100 rounded-md"
      >
        Join Room
      </button>
    </div>
  </div>
</template>

<script>
const AccessToken = require('twilio').jwt.AccessToken
const {
  connect,
  createLocalTracks,
  createLocalVideoTrack
} = require('twilio-video')
const VideoGrant = AccessToken.VideoGrant

export default {
  data: () => ({
    accessToken: '',
    localJoined: false,
    remoteJoined: false
  }),
  methods: {
    joinRoom() {
      connect(
        'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImN0eSI6InR3aWxpby1mcGE7dj0xIn0.eyJqdGkiOiJTSzMzNjllOGM1MTI1Mzk5OTY1OGNmM2QxZWQ1ZTYzOGIxLTE2MDE3NzcxMDkiLCJpc3MiOiJTSzMzNjllOGM1MTI1Mzk5OTY1OGNmM2QxZWQ1ZTYzOGIxIiwic3ViIjoiQUNhZTE2NjU4YjQ0YWNhMTYyOTlhZmQwYWM0N2UyYWYzOSIsImV4cCI6MTYwMTc4MDcwOSwiZ3JhbnRzIjp7ImlkZW50aXR5IjoiZXhhbXBsZS11c2VyIiwidmlkZW8iOnsicm9vbSI6IlRlc3QifX19.oHm_UfcNEvNyo8I3cSJ6fMio2MANOJTBTwZqpzlvRy4',
        {
          audio: true,
          name: 'Test',
          video: { width: 640 }
        }
      ).then(room => {
        console.log(`Connected to Room: ${room.name}`)
        this.localJoined = true

        const videoChatWindow = this.$refs.videoChatWindow

        console.log(videoChatWindow)
        createLocalVideoTrack().then(track => {
          videoChatWindow.appendChild(track.attach())
        })
        room.on('participantConnected', participant => {
          console.log(`Participant "${participant.identity}" connected`)

          participant.tracks.forEach(publication => {
            if (publication.isSubscribed) {
              const track = publication.track
              videoChatWindow.appendChild(track.attach())
            }
          })

          participant.on('trackSubscribed', track => {
            videoChatWindow.appendChild(track.attach())
          })
        })
      })
    }
  },
  mounted() {
    const twilioAccountSid = 'AC669496069f297202e6526c6428896165'
    const twilioApiKey = 'SK51eb2b458199990b8f7ee00cbe6065d7'
    const twilioApiSecret = '0bQmCHNJUEFvgvXvQAT7gzT8nsLKaU1D'
    const token = new AccessToken(
      twilioAccountSid,
      twilioApiKey,
      twilioApiSecret
    )
    token.identity = Math.random()
      .toString(20)
      .substr(2, 6)
    const videoGrant = new VideoGrant({
      room: 'Test'
    })
    // Add the grant to the token
    token.addGrant(videoGrant)

    this.accessToken = token.toJwt()
  }
}
</script>

<style></style>
