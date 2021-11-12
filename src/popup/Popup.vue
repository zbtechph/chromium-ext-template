<template>
    <div>
        <div>
            <h3>ZFlix v0.1</h3>
            <p>by: zbyte@phcorner</p>
        </div>
        <div>
            <div>
                <div v-if="!state.activeToken">
                    <input type="text" id="tokenTxt" placeholder="Enter Token" v-model="state.form.token"/>
                    <button id="importBtn" type="button" @click="importBtnClick">IMPORT</button>
                </div>
                <div v-else>
                    <p>{{ state.activeToken.description }}</p>
                    <button id="importBtn" type="button" @click="clearBtnClick">CLEAR</button>
                </div>
            </div>
            <div class="">
                <button id="exportBtn" @click="exportBtnClick">Export</button>
            </div>         
        </div>
    </div>
</template>

<script setup>
import { onMounted, reactive } from 'vue'

const state = reactive({
    form: {
        token: "",
    },
    activeToken: null
})

onMounted(()=>{
    chrome.storage.local.get(['activeToken'], result => {
        state.activeToken = result.activeToken ? result.activeToken : null
    })
})

async function importBtnClick(){
    if( ! state.form.token){
        return;
    }
    
    const response = await fetch(`https://soft-sunset-9cf2.zbph.workers.dev/${state.form.token}`)
    const data = await response.json();
    if(data){
        if(data.cookies.length === 0){
            alert("empty cookies!")
            return;
        }
        clearCookie();
        state.activeToken = data;
        await chrome.storage.local.set({ activeToken: data })
        data.cookies.forEach( async ( cookie ) => {
            const { domain, expirationDate, httpOnly, name, path, sameSite, secure, storeId, value } = cookie;
            const protocol = secure ? "https:" : "http:";
            const cookieUrl = `${protocol}//${domain.substr(1, domain.length)}${path}`;
            console.log(cookieUrl);
            await chrome.cookies.set({ domain, expirationDate, httpOnly, name, path, sameSite, secure, storeId, value, url: cookieUrl })
        })
    }
}

async function clearBtnClick(){
    state.activeToken = false;
    await chrome.storage.local.set({ activeToken: null })
    clearCookie();
}

async function exportBtnClick(){
    const url = new URL("https://netflix.com")
    const r = confirm("Export netflix cookie?")
    if( r ){
        const description = prompt("Description");
        const cookies = await chrome.cookies.getAll({ domain: url.hostname })
        if(cookies.length === 0){
            alert("cookie is empty!")
            return;
        }
        const response = await fetch("https://soft-sunset-9cf2.zbph.workers.dev/", {
            method: "POST",
            headers: {
                accept: "application/json",
                "content-type": "application/json"
            },
            body: JSON.stringify({
                description,
                cookies
            })
        })
        const data = await response.json()
        prompt(`${cookies.length} cookies has been sent!`, data.key);
    }
    
}

async function clearCookie(){
    const url = new URL("https://netflix.com")
    const cookies = await chrome.cookies.getAll({ domain: url.hostname })
    if(cookies.length === 0){
        return true;
    }

    const pending = cookies.map(cookie => {
        const protocol = cookie.secure ? "https:" : "http:";
        const cookieUrl = `${protocol}//${cookie.domain}${cookie.path}`;
        return chrome.cookies.remove({
            url: cookieUrl,
            name: cookie.name,
            storeId: cookie.storeId,
        });
    })
    await Promise.all(pending);
    return true;
}
</script>