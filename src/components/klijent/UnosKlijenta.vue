<template>
  <el-form label-width="120px">
    <el-form-item label="PIB">
        <el-input v-model="pib"></el-input>
    </el-form-item>
    <el-form-item label="Naziv klijenta">
        <el-input v-model="nazivKlijenta"></el-input>
    </el-form-item>
    <el-form-item label="Telefon">
        <el-input v-model="telefon"></el-input>
    </el-form-item>
    <el-form-item label="E-mail">
        <el-input v-model="email"></el-input>
    </el-form-item>
    <el-form-item label="Web strana">
        <el-input v-model="webStrana"></el-input>
    </el-form-item>
    <el-form-item label="Godina osnivanja">
        <el-input v-model="godinaOsnivanja"></el-input>
    </el-form-item>
    <el-form-item class="mesto" label="Odaberite mesto">
        <select @change="handleCitySelected" v-model="mesto" class="select">
            <option
                v-for="var_mesto in mesta"
                :key="var_mesto.postalCode"
                :label="var_mesto.name"
                :value="var_mesto"
            >
            </option>
        </select>
    </el-form-item>
    <el-form-item class="adresa" label="Odaberite ulicu i broj">
        <el-col class="adresa-first" :span="50">
            <select v-model="ulica" class="select">
                <option value="" disabled selected hidden>Ulica</option>
                <option
                    v-for="var_ulica in ulice"
                    :key="var_ulica.id"
                    :label="var_ulica.street"
                    :value="var_ulica"
                >
                </option>
            </select>
        </el-col>
        <el-col class="adresa-second" :span="50">
            <el-input v-model="broj" placeholder="Broj"></el-input>
        </el-col>
    </el-form-item>
    <el-form-item class="delatnost" label="Odaberi sifru delatnosti">
        <select v-model="sifraDelatnosti" class="select">
            <option
                v-for="var_delatnost in delatnosti"
                :key="var_delatnost.code"
                :label="var_delatnost.name"
                :value="var_delatnost"
            >
            </option>
        </select>
    </el-form-item>
    <el-form-item class="klijent" label="Odaberi potencijalnog klijenta">
        <select v-model="potencijalniKlijent" class="select">
            <option
                v-for="var_klijent in potencijalniKlijenti"
                :key="var_klijent.id"
                :label="var_klijent.name"
                :value="var_klijent"
            >
            </option>
        </select>
    </el-form-item>
    <!-- <el-form-item label="Ulica">
        <el-input v-model="ulica"></el-input>
    </el-form-item> -->
    
    <el-form-item>
        <el-button type="primary" @click="submitForm()">Unesi</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
import { reactive, ref, toRefs } from '@vue/reactivity'
import { ElNotification } from 'element-plus'
export default {
    setup() {
        const form = reactive({
          pib: '',
          nazivKlijenta: '',
          telefon: '',
          email: '',
          webStrana: '',
          godinaOsnivanja: '',
          ulica: '',
          broj: '',
          sifraDelatnosti: '',
          potencijalniKlijent: '',
          mesto: undefined,
        })

        const mesta = ref([])
        const ulice = ref([])
        const delatnosti = ref([])
        const potencijalniKlijenti = ref([])

        const postClientObjectTemplate = {
            pib: '',
            name: '',
            email: '',
            webPage: '',
            phoneNumber: '',
            yearOfEstablishment: '',
            address: {
                id: 0,
                number: 0
            },
            city: {
                postalCode: 0
            },
            potentialClient: {
                id: 0
            },
            activity: {
                code: 0
            }
        }


        fetch('http://localhost:8080/vip/api/city')
            .then((response) => response.json())
            .then((data) => { 
                console.log(data)
                mesta.value = data._embedded.city
            })
            .catch((error) => console.log(error))

        fetch('http://localhost:8080/vip/api/activity')
            .then((response) => response.json())
            .then((data) => {
                delatnosti.value = data._embedded.activity
            })
            .catch((error) => console.log(error))

        
        fetch('http://localhost:8080/vip/api/potentialClient')
            .then((response) => response.json())
            .then((data) => {
                console.log(data) 
                potencijalniKlijenti.value = data._embedded.potentialClient
            })
            .catch((error) => console.log(error))
    
        const handleCitySelected = () => {
            fetch(`http://localhost:8080/vip/client/address/${form.mesto.postalCode}`)
                .then((response) => response.json())
                .then((data) => {
                    ulice.value = data
                })
                .catch((error) => console.log(error))
        }

        const submitForm = () => {
            console.log(JSON.parse(JSON.stringify(form)))
            postClientObjectTemplate.pib = form.pib
            postClientObjectTemplate.name = form.nazivKlijenta
            postClientObjectTemplate.webPage = form.webStrana
            postClientObjectTemplate.phoneNumber = form.telefon
            postClientObjectTemplate.email = form.email
            postClientObjectTemplate.yearOfEstablishment = parseInt(form.godinaOsnivanja)
            postClientObjectTemplate.address.id = form.ulica.id
            postClientObjectTemplate.address.number = parseInt(form.broj)
            postClientObjectTemplate.city.postalCode = form.mesto.postalCode
            postClientObjectTemplate.potentialClient.id = form.potencijalniKlijent.id
            postClientObjectTemplate.activity.code = form.sifraDelatnosti.code
            console.log(postClientObjectTemplate)
            fetch('http://localhost:8080/vip/api/client', {
                method: 'POST',
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify(postClientObjectTemplate)
            })
                .then((response) => response.text())
                .then((data) => {
                    console.log(data)
                    ElNotification({
                        title: data,
                        message: 'You have succesfully saved a new request!',
                        type: 'success',
                    })
                })
                .catch((error) => {
                    console.log(error)
                    ElNotification({
                        title: 'FAIL',
                        message: 'You have succesfully saved a new request!',
                        type: 'success',
                    })
                })
        }

        return {
            ...toRefs(form),
            submitForm,
            handleCitySelected,
            mesta,
            ulice,
            delatnosti,
            potencijalniKlijenti,
        }
    }
}
</script>

<style scoped>

.mesto select {
    width: 100%;
}

.adresa .adresa-first {
    width: 50%;
}

.adresa .adresa-first select {
    width: 90%;
}

.adresa .adresa-second {
    width: 50%;
}

.delatnost select {
    width: 100%;
}

.klijent select {
    width: 100%;
}
</style>
