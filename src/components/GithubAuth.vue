<template>
  <div id="githubAuth">
    <v-btn style="background-color:#4e535b" v-if="this.$store.state.auth.discordAuthenticated && !this.$store.state.auth.githubAuthenticated" href="https://github.com/login/oauth/authorize?client_id=50c9f2e9a129a6bd1637&redirect_uri=https://www.macho.ninja:8000/api/githubauth/access_token&scope=user"><i class="fab fa-github"/>&nbsp;<span>Link Github Account</span></v-btn>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data: () => ({}),
  created () {
    const id = window.localStorage.getItem('githubId')
    const user = window.localStorage.getItem('githubUser')

    if (id && user) {
      this.$store.commit('setGithubId', id)
      this.$store.commit('setGithubUser', JSON.parse(user))
      this.$store.commit('setGithubAuthenticated', true)
    } else if (id) {
      this.fetchUser()
    } else if (!this.$store.state.auth.githubId && !this.$store.state.auth.githubUser && this.$route.query.githubId) {
      window.localStorage.setItem('githubId', this.$route.query.githubId)
      this.$store.commit('setGithubId', this.$route.query.githubId)
      this.fetchUser()
    }
  },
  methods: {
    unlink () {
      window.localStorage.removeItem('githubId')
      window.localStorage.removeItem('githubUser')
      this.$store.commit('setGithubAuthenticated', false)
      this.$store.commit('setGithubId', null)
      this.$store.commit('setGithubUser', null)
    },
    async fetchUser () {
      const { data: json } = await axios.get(`https://api.github.com/user?access_token=${window.localStorage.githubId}`)
      const { data: apiUser } = await axios.get(`https://www.macho.ninja/api/users/${this.$store.state.auth.discordUser.id}/links`)

      console.log('Fetched user')

      // eslint-disable-next-line
      if (apiUser.github.username == json.login) {
        console.log('User is already linked.')
        this.$store.commit('setGithubUser', json)
        window.localStorage.setItem('githubUser', JSON.stringify(json))
        this.$store.commit('setGithubId', json.login)
        window.localStorage.setItem('githubId', json.login)

        if (window.location.toString().includes('macho')) {
          window.location = 'https://www.macho.ninja'
        } else if (window.location.toString().includes('localhost')) {
          window.location = 'https://localhost:8080'
        }
        return true
      } else {
        console.log(`${apiUser.github.username} != ${json.login}`)
      }

      const linkRes = await axios.post(
        `https://www.macho.ninja/api/githubauth/link?discordId=${
          this.$store.state.auth.discordUser.id}&githubId=${json.login}&jwt=${window.localStorage.jwt || this.$store.state.auth.jwt}`)

      console.log(linkRes.data)

      if (linkRes.data.error) {
        this.$store.commit('setGithubId', null)
        this.$store.commit('setGithubAuthenticated', false)
        window.localStorage.removeItem('githubId')
        window.localStorage.removeItem('githubUser')

        console.log('Error')

        if (window.location.toString().includes('macho')) {
          window.location = 'https://www.macho.ninja'
        } else if (window.location.toString().includes('localhost')) {
          window.location = 'https://localhost:8080'
        }

        return
      }

      console.log('Linked account')

      if (window.location.toString().includes('macho')) {
        window.location = 'https://www.macho.ninja'
      } else if (window.location.toString().includes('localhost')) {
        window.location = 'https://localhost:8080'
      }
    }
  }
}
</script>
