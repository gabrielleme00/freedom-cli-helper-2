<template>
  <v-container class="fill-height" max-width="1000">
    <div class="w-100">
      <div class="mb-4 text-center">
        <h1 class="text-h2 font-weight-bold text-primary mb-2">Freedom CLI Helper</h1>
        <p class="text-h6 text-medium-emphasis">Comandos de inicialização</p>
      </div>

      <v-card class="mb-4" elevation="3" rounded="lg">
        <v-card-title class="bg-primary text-white">
          <v-icon start>mdi-cog</v-icon>
          Configuração
        </v-card-title>
        <v-card-text class="pa-6">
          <v-row>
            <v-col cols="12" md="6">
              <v-autocomplete
                v-model="selectedStore"
                :items="stores"
                density="comfortable"
                item-title="name"
                item-value="id"
                label="Loja"
                variant="outlined"
                prepend-inner-icon="mdi-store"
                placeholder="Digite para buscar uma loja..."
                return-object
                clearable
                no-data-text="Nenhuma loja encontrada"
              >
                <template v-slot:item="{ props }">
                  <v-list-item v-bind="props"></v-list-item>
                </template>
              </v-autocomplete>
            </v-col>
            
            <v-col cols="12" md="6">
              <v-autocomplete
                v-model="selectedMode"
                :items="modes"
                density="comfortable"
                item-title="value"
                item-value="value"
                label="Modo"
                variant="outlined"
                prepend-inner-icon="mdi-play-circle"
                placeholder="Digite para buscar um modo..."
                return-object
                clearable
                no-data-text="Nenhum modo encontrado"
              >
                <template v-slot:item="{ props, item }">
                  <v-list-item v-bind="props">
                    <template v-slot:prepend>
                      <v-icon>{{ item.raw.value === 'local' ? 'mdi-laptop' : 'mdi-cloud' }}</v-icon>
                    </template>
                    <v-list-item-subtitle>{{ item.raw.desc }}</v-list-item-subtitle>
                  </v-list-item>
                </template>
              </v-autocomplete>
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>

      <div v-if="selectedStore && selectedMode">
        <v-card elevation="3" rounded="lg">
          <v-card-title class="bg-success text-white">
            <v-icon start>mdi-console</v-icon>
            Comandos
          </v-card-title>
          <v-card-text class="pa-6">

            <div v-if="selectedMode.value === 'local'" class="mb-4">
              <v-card class="bg-grey-darken-3">
                <v-card-title class="text-h6 text-grey-lighten-2">
                  <v-icon start>mdi-server</v-icon>
                  1. Iniciar BFF Local
                </v-card-title>
                <v-card-text>
                  <p class="mb-3 text-medium-emphasis">
                    Execute este comando para iniciar seu servidor BFF local:
                  </p>
                  <v-card class="bg-grey-lighten-4 pa-3" variant="flat">
                    <v-btn 
                      icon="mdi-content-copy" 
                      size="small" 
                      variant="text" 
                      class="ml-2"
                      @click="copyToClipboard(bffCommand)"
                    ></v-btn>
                    <code class="text-body-1">{{ bffCommand }}</code>
                  </v-card>
                </v-card-text>
              </v-card>
            </div>

            <div class="mb-4">
              <v-card class="bg-grey-darken-3">
                <v-card-title class="text-h6 text-grey-lighten-2">
                  <v-icon start>mdi-console</v-icon>
                  {{ selectedMode.value === 'local' ? '2.' : '1.' }} Iniciar CLI
                </v-card-title>
                <v-card-text>
                  <p class="mb-3 text-medium-emphasis">{{ getCliDescription() }}</p>
                  <v-card class="bg-grey-lighten-4 pa-3" variant="flat">
                    <v-btn 
                      icon="mdi-content-copy" 
                      size="small" 
                      variant="text" 
                      class="ml-2"
                      @click="copyToClipboard(getCliCommand(selectedStore, selectedMode))"
                    ></v-btn>
                    <code class="text-body-1">{{ getCliCommand(selectedStore, selectedMode) }}</code>
                  </v-card>
                </v-card-text>
              </v-card>
            </div>

            <div>
              <v-card class="bg-grey-darken-3">
                <v-card-title class="text-h6 text-grey-lighten-2">
                  <v-icon start>mdi-eye</v-icon>
                  {{ selectedMode.value === 'local' ? '3.' : '2.' }} Iniciar View
                </v-card-title>
                <v-card-text>
                  <p class="mb-3 text-medium-emphasis">
                    Este comando iniciará o projeto View para gerar um novo
                    bundle quando houver uma alteração no código:
                  </p>
                  <v-card class="bg-grey-lighten-4 pa-3" variant="flat">
                    <v-btn 
                      icon="mdi-content-copy" 
                      size="small" 
                      variant="text" 
                      class="ml-2"
                      @click="copyToClipboard(getViewCommand(selectedStore))"
                    ></v-btn>
                    <code class="text-body-1">{{ getViewCommand(selectedStore) }}</code>
                  </v-card>
                </v-card-text>
              </v-card>
            </div>

          </v-card-text>
        </v-card>
      </div>
      
      <v-card v-else class="text-center pa-8" elevation="2" rounded="lg">
        <v-icon size="64" color="grey-lighten-1" class="mb-4">mdi-information-outline</v-icon>
        <h3 class="text-h5 text-medium-emphasis mb-2">Selecione Loja e Modo</h3>
        <p class="text-body-1 text-medium-emphasis">Escolha tanto uma loja quanto um modo acima para ver os comandos disponíveis</p>
      </v-card>

      <v-snackbar
        v-model="snackbar"
        :timeout="3000"
        color="success"
        rounded="pill"
      >
        <v-icon start>mdi-check-circle</v-icon>
        Comando copiado para a área de transferência!
      </v-snackbar>
    </div>
  </v-container>
</template>

<script setup>
  import { ref, watch, onMounted } from 'vue'

  const selectedStore = ref(null)
  const selectedMode = ref(null)
  const snackbar = ref(false)

  const stores = [
    { name: 'Allianz', id: 'L_ALLIANZPARQUE', dist: 'allianzparque' },
    { name: 'Clube Netshoes', id: 'L_CLUBENETSHOES', dist: 'clubenetshoes' },
    { name: 'ComSchool', id: 'L_COMSCHOOL', dist: 'comschool' },
    { name: 'Estante Virtual', id: 'L_ESTANTEVIRTUAL', dist: 'estantevirtual' },
    { name: 'Fluxo', id: 'L_FLUXO', dist: 'fluxo' },
    { name: 'Freelace', id: 'L_FREELACE', dist: 'freelace' },
    { name: 'Gap', id: 'L_GAP', dist: 'gapbrasil' },
    { name: 'Kappa', id: 'L_KAPPA', dist: 'kappa' },
    { name: 'Loja da Chape', id: 'L_CHAPECOENSE', dist: 'lojadachape' },
    { name: 'Loja do Inter', id: 'L_INTERNACIONAL', dist: 'lojadointer' },
    { name: 'NBA', id: 'L_NBA', dist: 'zattini' },
    { name: 'Netshoes', id: 'L_NETSHOES', dist: 'netshoesbr' },
    { name: 'Nerdstore', id: 'L_NERDSTORE', dist: 'nerdstore' },
    { name: 'NFL', id: 'L_NFL', dist: 'nflshop' },
    { name: 'Rainha', id: 'L_RAINHA', dist: 'rainha' },
    { name: 'Santos', id: 'L_SANTOS', dist: 'santosstore' },
    { name: 'SP Mania', id: 'L_SAOPAULO', dist: 'saopaulomania' },
    { name: 'Shoestock', id: 'L_SHOESTOCK', dist: 'shoestock' },
    { name: 'Shop Cruzeiro', id: 'L_CRUZEIRO', dist: 'shopcruzeiro' },
    { name: 'Shop Timão', id: 'L_CORINTHIANS', dist: 'shoptimao' },
    { name: 'Shop Vasco', id: 'L_VASCO', dist: 'shopvasco' },
    { name: 'Topper', id: 'L_TOPPER', dist: 'topper' },
    { name: 'WSL', id: 'L_WSL', dist: 'wslstore' },
    { name: 'Zattini', id: 'L_ZATTINI', dist: 'zattini' },
  ]

  const modes = [
    { value: 'local', desc: 'Usa o BFF rodando na sua máquina via localhost' },
    { value: 'remote', desc: 'Disponibiliza o seu bundle local em HMG via parâmetro' },
  ]

  const bffCommand = 'npm run start:dev'

  onMounted(() => {
    const savedStore = localStorage.getItem('freedom-cli-selected-store')
    const savedMode = localStorage.getItem('freedom-cli-selected-mode')
    
    if (savedStore) {
      try {
        const storeData = JSON.parse(savedStore)
        const foundStore = stores.find(store => store.id === storeData.id)
        if (foundStore) {
          selectedStore.value = foundStore
        }
      } catch (err) {
        console.error('Error parsing saved store:', err)
      }
    }
    
    if (savedMode) {
      try {
        const modeData = JSON.parse(savedMode)
        const foundMode = modes.find(mode => mode.value === modeData.value)
        if (foundMode) {
          selectedMode.value = foundMode
        }
      } catch (err) {
        console.error('Error parsing saved mode:', err)
      }
    }
  })

  watch(selectedStore, (newStore) => {
    if (newStore) {
      localStorage.setItem('freedom-cli-selected-store', JSON.stringify(newStore))
    } else {
      localStorage.removeItem('freedom-cli-selected-store')
    }
  }, { deep: true })

  watch(selectedMode, (newMode) => {
    if (newMode) {
      localStorage.setItem('freedom-cli-selected-mode', JSON.stringify(newMode))
    } else {
      localStorage.removeItem('freedom-cli-selected-mode')
    }
  }, { deep: true })

  const getCliCommand = (store, mode) => {
    return `npm run cli:${mode.value} -- -id ${store.id} -s ${store.dist}`
  }

  const getViewCommand = (store) => {
    return `npm run dev --store=${store.dist}`
  }

  const getCliDescription = () => {
    if (selectedMode.value?.value === 'local') {
      return 'Execute este comando para usar o seu BFF local:'
    }
    return 'Execute este comando para acessar seu bundle em HMG:'
  }

  const copyToClipboard = async (text) => {
    try {
      await navigator.clipboard.writeText(text)
      snackbar.value = true
    } catch (err) {
      console.error('Falha ao copiar o comando: ', err)
    }
  }

</script>
