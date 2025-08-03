# armips-lzss
[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/leomontenegro6/armips-lzss/blob/master/README.md)
[![pt-br](https://img.shields.io/badge/lang-pt--br-green.svg)](https://github.com/leomontenegro6/armips-lzss/blob/master/README.pt-br.md)

Esta é uma versão moodificada da ferramenta [armips assembler](https://github.com/Kingcom/armips), originalmente criada por Kingcom, e forkada pelo denim para nela adicionar novas funcionalidades.

A principal mudança é a inclusão de uma rotina nativa de compressão de dados em LZSS 0x10, que basicamente permite comprimir e inserir dados com compressão de bios de GBA numa passada só. Isso é muito útli para várias traduções e romhacks de GBA por aí, já que não há mais necessidade de manualmente comprimir os dados com ferramentas externas.

# Como usar

O uso da ferramenta é largamente o mesmo descrito [no repositório original](https://github.com/Kingcom/armips?tab=readme-ov-file#11-usage), com a adição da nova diretiva abaixo:

```
.lz77gba "snes-tomou-fumo.bin"
```

A diretiva acima funciona de forma muito similar a uma chamada de `.incbin` onde você passa o caminho de um arquivo específico para ser inserido na ROM, exceto que os dados são primeiro comprimidos em LZSS 0x10, e apenas depois inserido.

Por exemplo, imagine que um determinado dado comprimido em LZSS 0x10 está no offset 0x7FC0D0 da ROM. Para inserir novos dados comprimidos nesse offset, o código abaixo deve funcionar:

```
.org 0x087FC0D0
    .lz77gba "Graficos/snes-tomou-fumo-no-mesmo-offset.bin"
```

Mas geralmente, dados comprimidos acabam maiores do que o original, então nesta situação talvez seja preciso mover os dados comprimidos pro final da ROM, e atualizar seu ponteiro de acordo. Nesse cenário, assumindo que o ponteiro absoluto está no offset 0x02262C, o dado pode ser inserido dessa maneira:

```
; Catalogando ponteiro na macro "grafico1", para uso posterior.
.org 0x0802262C
    .dw grafico1

; Inserindo dados comprimidos no final da ROM,
; e atualizando seu ponteiro de acordo.
.org 0x08800000
grafico1:
    .lz77gba "Graficos/snes-tomou-fumo-no-final-da-rom.bin"
    .align
```

O exemplo acima inserirá o gráfico no offset 0x800000, e automaticamente atualizará seu ponteiro para apontar pro novo local. Após isso, ele alinhará o cursor do arquivo para um endereço múltiplo de 4, para garantir que outros dados comprimidos sejam inseridos num offset par. Do contrário, bugs poderão ocorrer.

E por aí vai. Se você tiver múltiplos dados comprimidos para serem inseridos, basta continuar catalogando seus ponteiros em outras macros, e inseri-los na seção abaixo da catalogação dos ponteiros, usando múltiplas chamadas de `.lz77gba`.

# Disclaimer

Embora esta ferramenta esteja aqui disponibilizada, ela foi fornecida as-is. O código-fonte dela não está disponível, e nenhum suporte será dado além do que está descrito nesta documentação. Use-a por sua conta e risco.