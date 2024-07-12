<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue'
import Button from 'primevue/button'
import 'primeicons/primeicons.css'

const server = reactive({ status: '', active: false, publicIp: '', running: false })
const buttonMessage = computed(() => (server.active ? 'Stop Server' : 'Start Server'))
const icon = computed(() => 'pi ' + (server.active ? 'pi-stop' : 'pi-play'))
const busy = ref(false)
const serverLoading = computed(() => {
  let loading = true
  loading = ['Running', 'Stopped'].every((status) => status !== server.status)
  if (server.status === 'Running' && server.publicIp === '') {
    loading = true
  }

  return !(!loading || !busy.value)
})

onMounted(async () => {
  refreshServerInfo()
})

function delay(ms: number) {
  return new Promise((resolve) => setTimeout(resolve, ms))
}

const sendRequest = async (body: object, path: string, method: string) => {
  try {
    await fetch('https://yuai6u56z8.execute-api.eu-west-1.amazonaws.com/' + path, {
      method: method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(body)
    }).then(async (response) => {
      const readable = response.body

      if (readable === null || readable === undefined) {
        throw new Error('Could not get instance information.')
      }

      const reader = readable.getReader()

      let chunks = []
      let done, value

      while (!done) {
        ;({ done, value } = await reader.read())
        if (value) {
          chunks.push(value)
        }
      }

      // Concatenate all Uint8Array chunks into one Uint8Array
      const totalLength = chunks.reduce((acc, chunk) => acc + chunk.length, 0)
      const combinedChunks = new Uint8Array(totalLength)
      let position = 0
      for (const chunk of chunks) {
        combinedChunks.set(chunk, position)
        position += chunk.length
      }

      // Convert the combined Uint8Array to a string
      const result = new TextDecoder().decode(combinedChunks)
      busy.value = false
      return result
    })
  } catch (err) {
    console.error('Error starting instance', err)
  }
}

const refreshServerInfo = async () => {
  let instanceInfo = await getInstanceInfo()

  if (instanceInfo !== null && instanceInfo !== undefined) {
    const instanceInfoParsed = JSON.parse(instanceInfo)

    server.status = toTitleCase(instanceInfoParsed?.Reservations[0].Instances[0].State.Name)
    server.publicIp = instanceInfoParsed?.Reservations[0].Instances[0].PublicIpAddress

    if (server.status !== 'Running' || (server.status === 'Running' && server.publicIp === '')) {
      setTimeout(() => {
        refreshServerInfo()
      }, 2000)
    } else {
      server.active = server.status === 'Running'
      server.running = server.active === true
      return
    }
  }
}

const toTitleCase = (str: string) => {
  return str
    .toLowerCase()
    .split(' ')
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ')
}

const getInstanceInfo = async () => {
  try {
    return fetch('https://yuai6u56z8.execute-api.eu-west-1.amazonaws.com/server', {
      method: 'GET',
      headers: { 'Content-Type': 'application/json' }
    }).then(async (response) => {
      const readable = response.body

      if (readable === null || readable === undefined) {
        throw new Error('Could not get instance information.')
      }

      const reader = readable.getReader()

      let chunks = []
      let done, value

      while (!done) {
        ;({ done, value } = await reader.read())
        if (value) {
          chunks.push(value)
        }
      }

      // Concatenate all Uint8Array chunks into one Uint8Array
      const totalLength = chunks.reduce((acc, chunk) => acc + chunk.length, 0)
      const combinedChunks = new Uint8Array(totalLength)
      let position = 0
      for (const chunk of chunks) {
        combinedChunks.set(chunk, position)
        position += chunk.length
      }

      // Convert the combined Uint8Array to a string
      const result = new TextDecoder().decode(combinedChunks)
      return result
    })
  } catch (err) {
    console.error('Error starting instance', err)
  }
}

const startMinecraftServer = async () => {
  await sendRequest({ action: 'start' }, 'server/minecraft', 'POST')
}

const toggleServer = async () => {
  busy.value = true
  server.active = !server.active

  const body = { action: 'stop' }

  if (server.active) {
    body.action = 'start'
    server.status = 'Starting...'
  } else {
    server.publicIp = ''
  }

  await sendRequest(body, 'server', 'POST')

  if (server.active) {
    while (!server.running) {
      await refreshServerInfo()
      await delay(5000)
    }

    // Wait 20 seconds for AWS Instance to initialiase and prepare to receive commands from SSM
    setTimeout(() => {
      console.log('starting')
      startMinecraftServer()
    }, 20000)
  } else {
    body.action = 'stop'
    server.status = 'Shutting Down Minecraft Server'

    await sendRequest(body, 'server/minecraft', 'POST')

    await delay(5000)
    await sendRequest(body, 'server', 'POST')

    await delay(2000)
    refreshServerInfo()
    busy.value = false
  }
}
</script>

<template>
  <header>
    <div class="wrapper">
      <h1>Minecraft Server Starter</h1>
    </div>
  </header>
  <main>
    <section
      :class="{
        borderRed: !server.active,
        borderYellow: serverLoading,
        borderGreen: server.running
      }"
      class="server-details"
    >
      <h3>Server Details</h3>
      <p
        :class="{
          red: !server.active,
          green: server.active
        }"
      >
        Status: {{ server.status }}
      </p>
      <p v-if="server.active">IP: {{ server.active ? server.publicIp : '' }}</p>
    </section>
    <Button
      :disabled="serverLoading"
      class="button"
      :icon="icon"
      @click="toggleServer()"
      :label="buttonMessage"
      :severity="server.active ? 'danger' : ''"
    />
  </main>
</template>

<style scoped>
.red {
  color: rgb(233, 122, 122);
}

.green {
  color: rgb(122, 233, 146);
}

.button {
  width: 50%;
}

.borderRed {
  border: #db6161 2px solid;
}

.borderGreen {
  border: #61db67 2px solid;
}

.borderYellow {
  border: #d3db61 2px solid;
}

h1 {
  font-weight: 700;
  font-size: 4rem;
}

h3 {
  font-weight: 600;
  font-size: 1.6rem;
}

header {
  color: rgb(206, 196, 196);
  line-height: 1.5;
  max-height: 100vh;
  text-align: center;
}

main {
  color: rgb(238, 231, 231);
  display: flex;
  text-align: center;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
  justify-content: center;
}

.server-details {
  background: #000000aa;
  padding: 1rem;
  border-radius: 1rem;
}

.server-details > p {
  font-size: 1.2rem;
  font-weight: 600;
}

@media (max-width: 1024px) {
  main {
    padding-top: 3rem;
  }

  h1 {
    font-weight: 700;
    font-size: 2rem;
  }

  h3 {
    font-weight: 600;
    font-size: 1rem;
  }
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
    margin-left: 4rem;
  }
}
</style>
