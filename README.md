# Lucas-vital
Oi
Este fluxo de trabalho criará um contêiner do docker, publicará e implantará no Tencent Kubernetes Engine (TKE).
#
# Para configurar este fluxo de trabalho:
#
# 1. Certifique-se de que seu repositório contém a configuração necessária para seu cluster Tencent Kubernetes Engine,
#     incluindo deployment.yml, kustomization.yml, service.yml, etc.
#
# 2. Configure segredos em seu espaço de trabalho:
#     - TENCENT_CLOUD_SECRET_ID com ID secreto da nuvem Tencent
#     - TENCENT_CLOUD_SECRET_KEY com a chave secreta da nuvem Tencent
#     - TENCENT_CLOUD_ACCOUNT_ID com ID de conta Tencent Cloud
#     - TKE_REGISTRY_PASSWORD com senha de registro TKE
#
# 3. Altere os valores das variáveis ​​de ambiente TKE_IMAGE_URL, TKE_REGION, TKE_CLUSTER_ID e DEPLOYMENT_NAME (abaixo).

nome : Tencent Kubernetes Engine

em :
  lançamento :
    tipos : [criado]

# Variáveis ​​de ambiente disponíveis para todos os trabalhos e etapas neste fluxo de trabalho
env :
  TKE_IMAGE_URL : ccr.ccs.tencentyun.com/demo/mywebapp
  TKE_REGION : ap-guangzhou
  TKE_CLUSTER_ID : cls-mywebapp
  DEPLOYMENT_NAME : tke-test

empregos :
  setup-build-publish-deploy :
    nome : Configurar, construir, publicar e implantar
    roda em : ubuntu-mais recente
    ambiente : produção
    passos :

    - nome : Checkout
      usa : ações / checkout @ v2
      
    # Build
    - name : Build Docker image
      correr : |        
        docker build -t $ {TKE_IMAGE_URL}: $ {GITHUB_SHA}.
    - nome : Registro TKE de login
      correr : |
        docker login -u $ {{secrets.TENCENT_CLOUD_ACCOUNT_ID}} -p $ {{secrets.TKE_REGISTRY_PASSWORD}} $ {TKE_IMAGE_URL}
    # Envie a imagem do Docker para o TKE Registry
    - nome : Publicar
      correr : |
        docker push $ {TKE_IMAGE_URL}: $ {GITHUB_SHA}
    - nome : Configurar Kustomize
      correr : |
        curl -o kustomize --location https://github.com/kubernetes-sigs/kustomize/releases/download/v3.1.0/kustomize_3.1.0_linux_amd64
        chmod u + x ./kustomize
    - name : Configure ~ / .kube / config para conectar o cluster TKE
      usa : TencentCloud / tke-cluster-credential-action @ v1
      com :
        secret_id : $ {{secrets.TENCENT_CLOUD_SECRET_ID}}
        secret_key : $ {{secrets.TENCENT_CLOUD_SECRET_KEY}}
        tke_region : $ {{env.TKE_REGION}}
        cluster_id : $ {{env.TKE_CLUSTER_ID}}
    
    - nome : Mudar para o contexto TKE
      correr : |
        kubectl config use-context $ {TKE_CLUSTER_ID} -context-default
    # Implante a imagem Docker no cluster TKE
    - nome : Implementar
      correr : |
        ./kustomize editar imagem do conjunto $ {TKE_IMAGE_URL}: $ {GITHUB_SHA}
        ./kustomize build. | kubectl apply -f -
        implantação de status de lançamento de kubectl / $ {DEPLOYMENT_NAME}
        kubectl get services -o wide
© 2021 GitHub, Inc.
Termos
Privacidade
Segurança
Status
Docs
Entre em contato com o GitHub
Preços
API
Treinamento
Blog
