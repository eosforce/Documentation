# 开发基于Chrome scatter钱包的应用的说明和部署文档

# 使用git下载当前项目
    git clone https://github.com/eosforce/eosc_scatter

# 安装ABI文档说明
    #进入下载后的项目目录
    npm install -g pm2 
    npm install
    #启动服务
    ./start_production.sh


# nginx配置
    server {
          listen 9999;
          location /v1/ {
             #你的节点rpc端口
              proxy_pass http://18.182.6.202:8888;
              proxy_set_header Host            $host;
              proxy_set_header X-Forwarded-For $remote_addr;
          }
          location /v1/chain/get_abi {
              #上边启动ABI服务的host:port
              proxy_pass http://127.0.0.1:5000;
              proxy_set_header Host            $host;
              proxy_set_header X-Forwarded-For $remote_addr;
          }
      }


# 前端

提示:您也可以不配置服务器，使用当前demo的服务器配置，或者在直接在demo代码上做开发。这样您将只需要开发前端，不需要配置上述nginx, ABI服务


    cd eosc_scatter
    npm install
    #开发
    npm run dev
    #打包服务器
    npm run build // eosc_scatter/dist/ 目录将会有压缩后的应用


# 前端调用scatter的API


1.getIdentity, 获取scatter的eos对象，用户名，权限


    import Eos from 'eosjs' //安装eosjs, npm install git+https://github.com/eosforce/eosjs.git#821bea5
    const scatter = window.scatter;
    // 节点配置
    const network = {
        blockchain: 'eos',
        // 若是服务器使用了ssl证书，用https, 若是没用使用http
        protocol: 'https',
        // 节点服务器外围域名，上边配置的nginx服务器
        host: 'explorer.eosforce.io',
        // 上边配置的nginx端口
        port: 443,
        chainId: 'bd61ae3a031e8ef2f97ee3b0e62776d6d30d4833c8f7c1645c657b149151004b',
    }
    let identify = scatter.getIdentity({accounts:[network]});
    //获取到scatter配置d的account
    const account = identity.accounts.find(function(acc){
        return acc.blockchain === 'eos';
    });
    //account.authority 是permission
    //选择的用户名account.name
    //创建操作的配置
    let options = {
         authorization: account.name+'@'+account.authority,
         broadcast: true,
         sign: true
    }
    //获取eos对象
    let eos = scatter.eos(network, Eos,  options, "https");
    


2.vote，投票


    //获取到的scatter的eos对象
    eos.vote(account_name, bpname, toAsset(amount, tokenSymbol), permission)
            .then(data => {
                return {
                    is_error: false,
                    data
                };
            })
            .catch(err => {
                return {
                    is_error: true,
                    msg: err
                };
            });


3.transfer，转账


    //获取到的scatter的eos对象
    let token = await eos.contract(tokenSymbol === 'EOS' ? 'eosio' : 'eosio.token').then(token => { return token });
    
    token.transfer(account_name, to, toAsset(amount), memo, permission)
                    .then(data => {
                        return {
                            is_error: false,
                            data
                        }
                    });


4.unfreeze，解冻


    //获取到的scatter的eos对象
    eos.unfreeze(account_name, bpname, permission)
      .then(data => {
          return {
              is_error: false,
              data
          };
      })
      .catch(err => {
          return {
              is_error: true,
              msg: err
          };
      });
      


5.claim，领取分红


    //获取到的scatter的eos对象
    eos.claim(account_name, bpname, permission)
                    .then(data => {
                        return {
                            is_error: false,
                            data
                        };
                    })
                    .catch(err => {
                        return {
                            is_error: true,
                            msg: err
                        };
                    });

