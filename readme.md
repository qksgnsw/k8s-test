
bashrc 에 넣기
alias ka='kubectl apply -f'
alias kd='kubectl delete -f'
alias chns='kubectl config set-context --current --namespace'

alias kgp='kubectl get pods -o wide'
alias kgs='kubectl get services -o wide'
alias kga='kubectl get all -o wide'

source <(helm completion bash)
source <(kubectl completion bash)