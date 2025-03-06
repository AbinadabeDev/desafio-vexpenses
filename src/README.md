Descrição detalhada e técnica do que o código faz

O primeiro bloco de código define o provedor em que região os recursos serão criados. Neste caso especificamente, fala-se de us-east-1 (Costa Leste dos Estados Unidos), segundo a documentação da Amazon na região Norte da Virgínia.

O segundo e terceiro bloco de código define duas variáveis, a saber, projeto e candidato. Elas serão usadas para personalizar os recursos criados.

No quarto bloco de código temos o recurso tls_private_key que é o responsável por gerar uma chave privada de criptografia. Além do tipo em primeiro momento, contamos com a definição personalizada do recurso que passa a ter a referência de ec2_key para o referenciarmos em outras partes do código dentro do Terraform.
O tipo de criptografia usada será a RSA, amplamente utilizado para segurança em conexões de rede. Especifica-se também o tamanho da chave, sendo esta de 2048 bits, considerado um tamanho seguro mantendo equilíbrio e segurança.
Em um contexto prático, essa chave será usada para configurar acessos SSH ou conexões seguras com instâncias EC2 na AWS.

O quinto bloco de código faz referência a criação de um par de chaves AWS com o recurso aws_key_pair, sendo uma privada, utilizada localmente e não passível de compartilhamento e a segunda pública. usada para configurar o acesso à instância EC2.
O nome interno para identificação é ec2_key_pair. Já o nome do par de chaves será a junção da variável projeto com candidato seguidas por -key. Essa personalização ajuda a pontuar a nomear o par de chaves com base em projeto e candidato específicos.
Por fim temos a chave pública gerada pela chave privada (local) RSA de 2048 bits. A chave pública em questão é usada no formato OpenSSH, permitindo assim o acesso à instância EC2.

Com o sexto bloco de código descreve a criação de uma VPC na AWS. Em termos gerais o VPC é uma rede isolada dentro da nuvem da AWS, permitindo controlar de forma detalhada os recursos. Em uma objetiva explicação do código podemos apontar que aws_vpc criará uma VPC na AWS, nos permitindo criar instâncias EC2, databases e outros serviços da AWS.
O CIDR block = "10.0.0.0/16" por sua vez permite a obtenção de vários endereços IPs. De acordo com o iptp.net, chegamos a 65.534 possíveis endereços. Habilitamos o suporte a DNS, ou seja, poderemos usar nomes de domínio para resolver endereços IP dentro da PVC e conseguinte, habilitamos a atribuição de nomes de host DNS para instâncias EC2. Ao criar uma instância ela terá um único nome DNS exclusivo.
As tags são metadados utilizados que são associados aos recursos. Neste caso, as tags são usadas para atribuir um nome descritivo à VPC, que será uma combinação das variáveis projeto e candidato. Em ambientes maiores e complexos pela quantidade de ações e implementações no infra é algo mais do que essencial.

No sétimo bloco de código já se introduz a aws_subnet criando uma sub-rede dentro da VPC da AWS. A sub-rede vai dividir a rede maior da PVC em partes menores otimizando processos e isolando recursos de forma eficiente. Atribui-se ao recurso o nome main_subnet como já o fazemos por padrão em todos os recursos aplicados em nossas linhas de código.
A sub-rede será criada dentro da VPC. O vpc_id é o identificados da VPC que fora criada anteriormente. Ou seja, a sub-rede main_subnet será criada dentro da VPC main_vpc.
Com o cidr_block = "10.0.1.0/24" definimos o intervalo de endereços IP para a sub-rede. Diferentemente da primeira definição, aqui temos o endereço IP com 24 bits nos deixando com 256 opções disponíveis. Assim sendo, essa rede é uma subdivisão da rede maior da VPC.
A zona de disponibilidade também é definida neste bloco através do comando availability_zone = "us-east-1a" fazendo menção a região citada no primeiro bloco. Essas zonas de disponibilidade são data centers fisicamente separados, mas próximos uns dos outros. Assim sendo, a sub-rede será criada nessa mesma zona.
As tags seguem o mesmo formato do que antes já fora citado, agrupamos os resultados das duas variáveis declaradas no bloco dois para que possamos dar um nome descritivo à sub-rede.

O oitavo bloco tem o recurso "aws_internet_gateway" "main_igw". Cria-se um internet gateway que é o responsável por fornecer uma conexão entre a VPC e a internet pública. Logo associamos o vpc_id ao IGW, sendo este id o mesmo criado no bloco anterior. As tags seguem o mesmo padrão de associação aos recursos, sendo definidas com base nas duas variáveis.


No novo bloco de código configura-se a tabela de rotas chamada main_route_table para a VPC especificada no id do bloco 7. A tabela de rotas será associada à VPC que foi criada anteriormente e ela direcionará o tráfego dentro dessa VPC.
A rota padrão é configurada com o bloco CIDR 0.0.0.0/0 que corresponde a qualquer destino na internet. A parte gateway_id = aws_internet_gateway.main_igw.id define que qualquer tráfego fora da VPC será destinado para o Internet Gateway.

No décimo bloco conectamos a tabela de rotas à subnet permitindo que esta utilize a tabela para definir como o tráfego de rede será roteado. Após isso qualquer instância EC2 ou recurso seguirás as rotas configuradas na tabela de rotas.

No decimo primeiro abordamos a parte de segurança. A regra de entrada (ingress) permite que qualquer máquina independente do seu IP se conecte via SSH à instância EC2. A porta 22 é padrão para SSH, tornado possível que qualquer pessoa com a chave privada se conecte à instância.
A regra de saída (egress) permite que qualquer tráfego de saída seja enviado da instância para qualquer destino, incluindo chamadas para APIs externas, acesso à web ou comunicação com outras instâncias.
As configurações de segurança não são as mais recomendadas pois neste caso, as conexões via SSH estão sendo permitidas de qualquer lugar, o que pode ser um risco. Além disso nenhuma restrição de saída também é definida, o que configura mais um risco.

O décimo segundo bloco define o recurso que irá procurar a última AMI Debian 12 com base no Nome, Tipo de Visualização e Proprietário. debian-12-amd64-* garante uma imagem do Debian 12 para a arquitetura AMD64. A hvm é o tipo de visualização que oferece o melhor desempenho e a propriedade fica a cargo da AWS sob o seu id.

