ABAP_CODE_3
===========

ABAP program to generate an output txt file in your computer.
==========================

This ABAP program has been created for who is starting studying ABAP. The whole code is commented.

The code below will generate an output txt file. 
O código abaixo irá gerar um arquivo de saída txt.

INSTALLATION
------------

You may copy and paste the code in your ABAP enviorement to test. 

<div class><pre>

REPORT  zfii001_10.

TABLES: bkpf,
        bseg.

TYPES:
  BEGIN OF ty_bkpf,
  bukrs TYPE bkpf-bukrs,
  gjahr TYPE bkpf-gjahr,
  belnr TYPE bkpf-belnr,
  budat TYPE bkpf-budat,
  waers TYPE bkpf-waers,
  END OF ty_bkpf.

TYPES:
  BEGIN OF ty_bseg,
  bukrs TYPE bseg-bukrs,
  gjahr TYPE bseg-gjahr,
  belnr TYPE bseg-belnr,
  buzei TYPE bseg-buzei,
  hkont TYPE bseg-hkont,
  bschl TYPE bseg-bschl,
  shkzg TYPE bseg-shkzg,
  dmbtr TYPE bseg-dmbtr,
  END OF ty_bseg.

SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME.
SELECT-OPTIONS: s_bukrs FOR bkpf-bukrs OBLIGATORY,
                s_gjahr FOR bkpf-gjahr OBLIGATORY,
                s_belnr FOR bkpf-belnr.
SELECTION-SCREEN END OF BLOCK b1.

DATA: wa_bkpf TYPE ty_bkpf,
      it_bkpf TYPE STANDARD TABLE OF ty_bkpf,
      wa_bseg TYPE ty_bseg,
      it_bseg TYPE STANDARD TABLE OF ty_bseg,
      c_dmbtr TYPE c LENGTH 13,
      v_soma TYPE p,
      v_soma2 TYPE p LENGTH 8 DECIMALS 2,
      c_soma TYPE c LENGTH 10,
      c_soma2 TYPE c LENGTH 10.


DATA: BEGIN OF wa_texto,
      c_texto TYPE c LENGTH 200,
      END OF wa_texto.

DATA: BEGIN OF it_texto OCCURS 0,
      c_texto TYPE c LENGTH 200,
      END OF it_texto.

SELECT *
   FROM bkpf
   INTO CORRESPONDING FIELDS OF TABLE it_bkpf
   WHERE bukrs IN s_bukrs[]
     AND gjahr IN s_gjahr[]
     AND belnr IN s_belnr[].

SELECT *
  FROM bseg
  INTO CORRESPONDING FIELDS OF TABLE it_bseg
  WHERE bukrs IN s_bukrs[]
    AND gjahr IN s_gjahr[]
    AND belnr IN s_belnr[].

LOOP AT it_bkpf INTO wa_bkpf.

  APPEND 'HEADER' TO it_texto.

  CONCATENATE wa_bkpf-bukrs';'
              wa_bkpf-gjahr';'
              wa_bkpf-belnr';'
              wa_bkpf-budat';'
              wa_bkpf-waers';'
              INTO wa_texto-c_texto.

  APPEND wa_texto TO it_texto.

  APPEND 'ITEM' TO it_texto.

  LOOP AT it_bseg INTO wa_bseg
      WHERE bukrs EQ wa_bkpf-bukrs
        AND gjahr EQ wa_bkpf-gjahr
        AND belnr EQ wa_bkpf-belnr.

    c_dmbtr = wa_bseg-dmbtr.

    CONCATENATE wa_bseg-buzei';'
                wa_bseg-hkont';'
                wa_bseg-bschl';'
                wa_bseg-shkzg';'
                c_dmbtr';'
                INTO wa_texto-c_texto.
    v_soma = v_soma + 1.
    v_soma2 = v_soma2 + wa_bseg-dmbtr.
    APPEND wa_texto TO it_texto.
  ENDLOOP.
ENDLOOP.

c_soma = v_soma.
c_soma2 = v_soma2.

APPEND 'trailer' TO it_texto.
CONCATENATE c_soma';'
            c_soma2
            INTO wa_texto-c_texto.
APPEND wa_texto TO it_texto.

data v_url like rlgrap-filename.
*v_url = 'C:\Documents and Settings\Student\Desktop\Teste.txt'.
v_url = 'C:\Users\nspadm\Desktop\Teste.txt'.

CALL FUNCTION 'WS_DOWNLOAD'
  EXPORTING
    filename         = v_url
    filetype         = 'ASC'
  TABLES
    data_tab         = it_texto
  EXCEPTIONS
    file_open_error  = 1
    file_write_error = 2
    OTHERS           = 3.

CASE sy-subrc.
  WHEN 0.
    MESSAGE 'Gravado com sucesso' TYPE 'S'.
  WHEN 1.
    MESSAGE 'Nenhum informação encontrada' TYPE 'I' DISPLAY LIKE 'E'.
    STOP.
  WHEN 2.
    MESSAGE 'Erro na gravação do arquivo' TYPE 'I' DISPLAY LIKE 'E'.
    STOP.
  WHEN OTHERS.
    MESSAGE 'Try again!' TYPE 'I' DISPLAY LIKE 'E'.
ENDCASE.
