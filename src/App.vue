<template>
<div id="app">
     <div class="formNovoCartao">
      <form @submit.prevent="gerar" >
        <input type="text" placeholder="Nome Completo" id="nome" v-model="solicitacao.titular">
        <input type="number" step="0.010" placeholder="Informe o saldo" id="saldoInicial" v-model="solicitacao.saldo">
        <input type="submit" class="btn2" id="btnSolicita" value="Emitir Cartão">
      </form>
    </div>

  

<div class="cartaoForm"> 
        <div class="logo">
          <img src="./assets/credprepaylogo.png" height="50px" width="100%" alt="CredPrePay">
        </div>
        <div class="numCartao" id="container-cartao">
            <input type="text" class="input" id="numeroCartao"  placeholder="0000 0000 0000 0000" readonly="readonly" v-model="cartao.numero">
          </div>
    <div class="cartaoInfo">
      <div class="validade">
          <input id="mesValidadeCartao" type="text" class="validadeInput" placeholder="MM/YY" readonly="readonly" v-model="cartao.validade">
        </div>
      <div class="cvv">
        <input id="cvvCartao" type="text" class="cvvInput" placeholder="000" readonly="readonly" v-model="cartao.cvv">  
        <div class="cvvLogo">?
          <div class="logo">
            <img src="./assets/cvv.png" height="80px" width="80px" alt="CVV">
          </div>
        </div>
      </div>
    </div>
    
    <div class="titular">
        <input type="text" class="input" id="titularCartao" placeholder="NOME DO TITULAR" readonly="readonly" v-model="cartao.titular" >
    </div>  
</div>



<div v-show="retornoMessage" class="informacao" id="info">
</div>



<div class="formCompra">
      <form @submit.prevent="pagar" >
        <input type="text" placeholder="Estabelecimento" id="estabelecimeto" v-model="pagamento.estabelecimento">
        <input type="text" placeholder="Numero de Cartao" id="compraNumero" v-model="pagamento.numeroCartao">
        <input type="text" placeholder="Validade" id="compraValidade" v-model="pagamento.validade">
        <input type="text" placeholder="CVV" id="compraCVV" v-model="pagamento.cvv">
        <input type="text" placeholder="Senha" id="compraSenha" v-model="pagamento.senha">
        <input type="number" step="0.010" placeholder="Preço a pagar" id="compraValor" v-model="pagamento.valorCompra">
        <input type="submit" class="btnPagar" id="btnSolicita" value="Pagar">
      </form>
    </div>
</div>

</template>

<script>
//dá para componentizar muito mais cada item desse!!! 
import Cartao from './services/cartoes'

export default {
  data(){
    return{
      solicitacao: {
        titular: '',
        saldo: ''
      },
      cartoes:[],
      cartao: {},
      transacao: {},
      //retornoMessage: 'O cartão pré pago é uma excelente forma de pagamento! Solicite agora!',
      retornoMessage: '',
      pagamento:{
        estabelecimento: '',
        valorCompra: '',
        cvv:'',
        validade: '',
        numeroCartao: '',
        senha: ''

      }
      
    }
  },



  mounted() {
    
  },


  methods:{
      maskCC(v){
          v = v.replace(/\D/g,""); // Permite apenas dígitos
          v = v.replace(/(\d{4})/g, "$1 "); // Coloca um ponto a cada 4 caracteres
          v = v.replace(/\.$/, ""); // Remove o ponto se estiver sobrando
          return v;
      },
      
    gerar(){
      
      Cartao.gerarCartao(this.solicitacao).then(resposta=> {

      var ul = document.createElement('ul');
      var info = document.getElementById('info');
      var li = document.createElement('li');
      ul.setAttribute('id','ulInfo');
      ul.setAttribute('style','list-style: none;');
      info.innerHTML = '';

      this.solicitacao = {}
      this.cartao = resposta.data
      this.cartao.numero = this.maskCC(this.cartao.numero)  
      this.retornoMessage = `Parabéns ${this.cartao.titular.split(' ')[0]} 

      seu cartão de crédito pré-pago foi gerado! 
      O saldo disponível para compras é de R$ ${this.cartao.saldo} e sua senha é: ${this.cartao.senha}.`
      
      li.appendChild(document.createTextNode(this.retornoMessage));
      ul.appendChild(li);
      info.appendChild(ul);

      })
    },
    pagar(){
      var ul = document.createElement('ul');
      var info = document.getElementById('info');
      var li = document.createElement('li');
      ul.setAttribute('id','ulInfo');
      ul.setAttribute('style','list-style: none;');

      Cartao.pagarCompra(this.pagamento).then(resposta=> {
      
        this.retornoMessage = {}
        this.transacao = resposta.data

        if(this.transacao.autorizado == 'true'){
            info.innerHTML = '';
            this.retornoMessage = this.transacao.status
            li.appendChild(document.createTextNode(this.retornoMessage));
            ul.appendChild(li);    
            this.pagamento = {}
        }else{
          info.innerHTML = '';          
           var erroList = this.transacao.status.split("#");
            li.appendChild(document.createTextNode("Erro."));

                for (var i = 0; i < erroList.length; i++) {
                    li = document.createElement('li');
                    li.appendChild(document.createTextNode(erroList[i]));
                    ul.appendChild(li);                    
                }
        }
        info.appendChild(ul)
        this.retornoMessage = {}
        this.pagamento = {}
       })               
        
    }
  }

  
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
@import url('https://fonts.googleapis.com/css?family=Secular+One&display=swap');

*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  outline: none;
  font-family: 'Secular One', sans-serif;
}

body{
  background: rgb(255, 255, 255); 
  background-size: cover;
  width: 100%;
  height: 100%;
}

.formNovoCartao{
  display: flex;
  position: absolute;
  top: 10%;
  left: 50%;
  transform: translate(-50%, -50%);
  max-width: 610px;
  width: 100%;
  background: rgba(201, 73, 63, 0.5);
  padding: 30px;
  border-radius: 10px;
}

.formCompra{
  display: flex;
  position: absolute;
  top: 75%;
  left: 50%;
  transform: translate(-50%, -50%);
  max-width: 610px;
  width: 100%;
  background: rgba(201, 73, 63, 0.5);
  padding: 30px;
  border-radius: 10px;
}

.formCompra input{
  
  justify-content: space-between;
  margin-bottom: 10px;

  position: relative;
  margin-right: 50px;
  padding: 20px;
}

.informacao{
  display: flex;
  text-align: left;
  position: absolute;
  top: 25%;
  left: 50%;
  transform: translate(-50%, -50%);
  max-width: 610px;
  width: 100%;
  background: rgba(143, 194, 162, 0.5);
  padding: 30px;
  border-radius: 10px;
}





.btn{
  display: inline;
  background-color:green;
  color: rgb(255, 255, 255);
  text-align: center;
  border-radius: 3px;
  padding: 10px;
  text-transform: uppercase;
  font-size: 16px;
  cursor: pointer;
}



#btnSolicita{
  display: inline;
  background-color:green;
  color: rgb(255, 255, 255);
  text-align: center;
  border-radius: 5px;
  padding: 6px;
  text-transform: uppercase;
  font-size: 18px;
  cursor: pointer;
  top: 20%;

}

#btnPagar{
  display: inline;
  background-color:blue;
  color: rgb(255, 255, 255);
  text-align: center;
  border-radius: 5px;
  padding: 6px;
  text-transform: uppercase;
  font-size: 18px;
  cursor: pointer;
  top: 20%;

}



.formNovoCartao input{
  position: relative;
  margin-right: 35px;
  padding: 30px;
  

}



.cartaoForm{
  position: absolute;
  top: 48%;
  left: 50%;
  transform: translate(-50%, -50%);
  max-width: 380px;
  width: 100%;
  background: rgba(103, 25, 31, 100);
  padding: 30px;
  border-radius: 25px;

}

.retornoMessage{
  position: absolute;
  top: 60%;
  left: 25%;
  max-width: 50%;
  background: red;
  width: 50%;
  padding: 50px;
  border-radius: 15px;
}

.retornoMessage input{
  top: 60%;
  left: 25%;
  width: 50%;

  border-radius: 3px;
  font-size: 16px;
  border: 0;
  text-align: center;
}


.numCartao{
  margin-bottom: 10px;
}

input[type="text"]{
  padding: 2px;
  border-radius: 3px;
  font-size: 16px;
  border: 0;
  text-align: center;

}

input[type="number"]{
  padding: 2px;
  border-radius: 3px;
  font-size: 16px;
  border: 0;
  text-align: center;
}

.numCartao .input{
  width: 100%;
  text-align: center;
  font-size: 25px;
  background-color: transparent;
  color: #fff;

}

.cartaoInfo{
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.validade  .validadeInput{
  width: 150px;
  margin-right: 1px;
  background-color: transparent;
  color: #fff;

}

.cvv .cvvInput{
  background-color: transparent;
  width: 100px;
  margin-right: 10px;
  color: #fff;

}

.validade,
.cvv{
  display: flex;
}

.cvv{
  justify-content: flex-end;
  align-items: center;
}

.cvvLogo{
  border: 2px solid #fff;
  width: 25px;
  height: 25px;
  color: rgb(253, 253, 253);
  font-size: 18px;
  font-weight: bold;
  border-radius: 50%;
  text-align: center;
  position: relative;
  cursor: pointer;
}
.cvvLogo .logo{
  position: absolute;
  width: 90px;
  height: 75px;
  padding:  10px 10px 5px;
  border-radius: 3px;
  top:-30px;
  right: 0;
  background: rgba(247, 243, 243, 0.2);
  display: none;
}

.cvvLogo .logo .img{
  width: 100px;
  height: 100px;
}

.cvvLogo:hover .logo{
  display: block;
}

.titular{
  width: 100%;
}

.titular .input{
  width: 100%;
  text-align: center;
  font-size: 16px;
  background-color: transparent;
  text-transform:uppercase;
  color: #fff;

}

::-webkit-input-placeholder{
  color: #757575;
}
</style>
