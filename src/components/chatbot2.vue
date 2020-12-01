<template>
   <VueBotUI
    :options="options"
    :is-open="isOpen"
    :bot-typing="botTyping"
    :input-disable="inputDisable"
    :messages="messages"
    @msg-send="onSend"
    
   >

   </VueBotUI>
</template>
<script>
import {VueBotUI} from 'vue-bot-ui'
import axios from 'axios'
import {API_KEY,API_URL} from '../global'
export default {
    components :{
        VueBotUI
    },
    data : function()
    {
        return {
            options : {
                botTitle:'Mindler',
                botAvatarImg:require('@/assets/logo.jpg')
            },
            botTyping:false,
            isOpen:true,
            inputDisable:false,

           user:{
               name:null,
               email:null
           },
           
            nameRegex:/^[a-zA-Z ]+$/,
            emailRegex:/^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/,
            messages : [
                {agent:'bot',type:'text',text:"Welcome to the Mindler, India's No.1 site for Carrer Planning and Counsling"},               
                ]
        }
    },

    mounted()
    {  
        this.sendBotMessage('Okay! Please, tell me your name first.');          
    },
    methods : {
        onSend(v)
        {
            this.sendUserMessage(v.text) 

            if(this.user.name==null)
            {
               this.acceptName(v)
                return;
            }
            if(this.user.email==null)
            {
                this.acceptEmail(v)
                return; 
            }
            
             this.postback(v)
              
            
        },

        acceptName(v)
        {
           
            if(this.nameRegex.test(v.text))
            {
                this.user.name = v.text;
                this.sendBotMessage(`Howdy, ${this.user.name}! pls enter  your e-mail`);
            }
            else
            {
                this.sendBotMessage("Plese enter a valid name")
            } 
            
        },
        acceptEmail(v)
        {
            
            if(this.emailRegex.test(v.text))
            {
                this.user.email = v.text;
                 this.initChat();
            } 
            else
            {
                this.sendBotMessage("Please enter a valid email address");
                
            }
        },

        
       
        sendBotMessage(text)
        {         
                 this.messages.push( {
                    agent:'bot',
                    type:'text',
                    text:text
                })
        },
        sendUserMessage(text)
        {
             this.messages.push( {
                    agent:'user',
                    type:'text',
                    text:text
                })
        },
        initChat()
        {
            this.postback({value:'Hi'})
        },

        postback(v)
        {
            this.botTyping=true;
            var data = JSON.stringify({"API_key":API_KEY,"query":v.value});
        
            console.log(data);
            
            var config = {
            method: 'post',
            url: API_URL,
            headers: { 
                'Content-Type': 'application/json'
            },
            data : data
            };

            var ref =this;

            axios(config)
            .then(function (response) {
            //console.log(JSON.stringify(response.data));
            if(!response.data.Error)
            {
                var options = []

                response.data.Options.forEach(option => {
                    options.push({
                       text:option,
                       value:option,
                       action:'postback' 
                    })
                });
                
                
                
                var newMessage = {
                        agent: 'bot',
                        type: 'button',
                        text: response.data.Question,
                        disableInput: true,
                        options: options,
                }

                console.log(JSON.stringify(newMessage))

                ref.messages.push(newMessage)
            }

            })
            .catch(function (error) {
                    console.log('Error Reported',error);
                    ref.sendBotMessage('Sorry, I didn\'t get you. Try Again!');
                    ref.initChat();
                    
            })
            .finally(()=>{
                this.botTyping=false;
            })
           
        }
        
    }
    
}
</script>
<style lang="scss">

.qkb-board {
    width:400px !important;
}

.qkb-msg-bubble {
  display: flex;
  position: relative;
}
.qkb-board-content {
    flex-grow: 1;
    
}

.qkb-mb-button-options__btn 
{

  white-space: normal!important;
  position:relative; 
  margin: .5rem .375rem 0 0;
  padding: .25rem .5rem;
  cursor: pointer;
  outline: 0;
  border: 1px solid transparent;
  border-radius: 13px;
  color: inherit;
  font-size: 0.875rem;
  font-family: inherit;
  text-decoration: none;
  background-color: transparent;
  transition: background-color linear .15s, color linear .1s;
max-width: 306px !important;
  span {
    position: relative;
    z-index: 10;
    overflow: scroll !important;
    word-wrap: break-word !important;
    
  }
}
</style>