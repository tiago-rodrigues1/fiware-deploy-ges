# FIWARE + WireCloud

Este projeto inicia WireCloud, Orion-LD e IoT Agent JSON com Docker Compose. O WireCloud fica disponível em `http://localhost:8000`; o Orion-LD em `http://localhost:1026`; e as APIs do IoT Agent em `http://localhost:4041` (provisionamento) e `http://localhost:7896` (HTTP para dispositivos). O broker MQTT escuta em `localhost:1883`.

## Iniciar os serviços

Na raiz do projeto, execute:

```bash
docker compose up -d
```

Confira os containers:

```bash
docker compose ps
```

Na primeira inicialização, aguarde WireCloud e os demais serviços terminarem de iniciar. Para acompanhar o WireCloud:

```bash
docker compose logs -f wirecloud
```

Abra `http://localhost:8000` no navegador.

## Criar o superuser

Com o container `wirecloud` em execução, crie a conta administrativa:

```bash
docker exec -it wirecloud python manage.py createsuperuser
```

Informe o nome de usuário, e-mail e senha quando o comando solicitar. Depois, entre no WireCloud em `http://localhost:8000` com essas credenciais.

Se o usuário já existir e você precisar trocar a senha:

```bash
docker exec -it wirecloud python manage.py changepassword <usuario>
```

## Enviar pacotes `.wgt`

Os pacotes disponíveis na raiz deste projeto são:

- `CoNWeT_ngsi-source_4.2.0.wgt` — consulta entidades NGSI no Orion-LD.
- `CoNWeT_ngsientity2poi_3.2.2.wgt` — converte entidades em pontos de interesse (PoI).
- `CoNWeT_ol3-map_1.2.4.wgt` — exibe PoIs em um mapa OpenLayers.

Para enviar pela interface:

1. Entre no WireCloud.
2. Abra **My Resources**.
3. Selecione **Upload** e escolha os arquivos `.wgt` desejados.
4. Confirme o envio.

Também é possível publicar os pacotes para todos os usuários pelo terminal. O comando abaixo copia os arquivos para o container e os adiciona ao catálogo público:

```bash
for file in ./*.wgt; do
  docker cp "$file" wirecloud:/tmp/
done

docker exec wirecloud sh -c 'python manage.py addtocatalogue --public --redeploy /tmp/*.wgt'
```

O comando `--redeploy` atualiza no catálogo um pacote que já tenha sido enviado. O WireCloud documenta `addtocatalogue` como a ferramenta administrativa para incluir componentes no catálogo.

## Criar um mashup e fazer o wiring

1. No WireCloud, crie um **New workspace**.
2. Adicione ao canvas os componentes **NGSI source**, **NGSI Entity To PoI** e **OpenLayers Map**. Eles aparecem após os respectivos `.wgt` serem enviados.
3. Abra a vista **Wiring** pelo botão/ícone de conexões do editor.
4. Crie estas conexões entre as saídas e entradas:

   ```text
   NGSI source: Entities (entityOutput)
       → NGSI Entity To PoI: Entity (entityInput)

   NGSI Entity To PoI: PoI (poiOutput)
       → OpenLayers Map: Insert/Update PoI (poiInput)
   ```

5. Salve o workspace e volte à visualização do mashup.

No componente **NGSI Entity To PoI**, configure `coordinates_attr` para o atributo geográfico da entidade. O padrão é `location`; se suas entidades guardarem latitude e longitude separadas, informe os dois nomes separados por vírgula, por exemplo `latitude, longitude`.

No **NGSI source**, configure o servidor Orion como `http://localhost:1026/` quando estiver usando o navegador na mesma máquina do Docker. Ajuste também os filtros de tipo/ID e o tenant (`FIWARE-Service`) para corresponder às entidades. O operador só gera PoIs para entidades que tenham a localização configurada.

## Encerrar

Para parar os containers mantendo os dados nos volumes:

```bash
docker compose down
```

Os dados do PostgreSQL, MongoDB e WireCloud ficam em volumes Docker e continuam disponíveis após parar os containers.
