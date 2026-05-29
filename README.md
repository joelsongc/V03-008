

         break;

         case 2:

            if(bobina[bobina_atual].espiras < 9999)
               bobina[bobina_atual].espiras++;

         break;

         case 3:

            if(bobina[bobina_atual].bitola < 32)
               bobina[bobina_atual].bitola++;

         break;

         case 4:

            if(bobina[bobina_atual].inicio < 5000)
               bobina[bobina_atual].inicio++;

         break;

         case 5:

            if(bobina[bobina_atual].fim < 5000)
               bobina[bobina_atual].fim++;

         break;

         case 6:

            if(bobina[bobina_atual].velocidade < 10)
               bobina[bobina_atual].velocidade++;

         break;
      }
   }

   //------------------------------------------------------
   // DW
   //------------------------------------------------------

   if(key_press & KEY_DW){

      switch(menu_nivel){

         case 1:

            if(bobina_atual > 1)
               bobina_atual--;

         break;

         case 2:

            if(bobina[bobina_atual].espiras > 0)
               bobina[bobina_atual].espiras--;

         break;

         case 3:

            if(bobina[bobina_atual].bitola > 16)
               bobina[bobina_atual].bitola--;

         break;

         case 4:

            if(bobina[bobina_atual].inicio > 0)
               bobina[bobina_atual].inicio--;

         break;

         case 5:

            if(bobina[bobina_atual].fim > 0)
               bobina[bobina_atual].fim--;

         break;

         case 6:

            if(bobina[bobina_atual].velocidade > 1)
               bobina[bobina_atual].velocidade--;

         break;
      }
   }
}

//---------------------------------------------------------
// START + FALHA
//---------------------------------------------------------

void task_start(){

   //------------------------------------------------------
   // FALHA INVERSOR
   //------------------------------------------------------

   if(FAL == 0){

      falha_inversor = 1;

      maquina_run = 0;

      ONOFF = 0;

      FREIO = 1;

      return;
   }

   //------------------------------------------------------
   // FALHA NORMALIZADA
   //------------------------------------------------------

   falha_inversor = 0;

   //------------------------------------------------------
   // START
   //------------------------------------------------------

   if(menu_nivel == 0){

      //---------------------------------------------------
      // START MOMENTÂNEO
      //---------------------------------------------------

      if(STRT == 0){

         //------------------------------------------------
         // HABILITA RUN
         //------------------------------------------------

         maquina_run = 1;

         //------------------------------------------------
         // LIGA INVERSOR
         //------------------------------------------------

         ONOFF = 1;

         //------------------------------------------------
         // LIBERA FREIO
         //------------------------------------------------

         FREIO = 0;
      }
   }
}

//---------------------------------------------------------
// CONTAGEM
//---------------------------------------------------------

void task_encoder(){

   //------------------------------------------------------
   // SOMENTE EM RUN
   //------------------------------------------------------

   if(maquina_run == 0){

      sa_old = SA;

      return;
   }

   //------------------------------------------------------
   // BORDA DE SUBIDA
   //------------------------------------------------------

   if(SA == 1 && sa_old == 0){

      contador_espiras++;

      if(contador_espiras > 9999)
         contador_espiras = 9999;
   }

   //------------------------------------------------------
   // MEMÓRIA
   //------------------------------------------------------

   sa_old = SA;
}

//---------------------------------------------------------
// DISPLAY
//---------------------------------------------------------

void task_display(){

   //------------------------------------------------------
   // AUTO RECOVERY LCD
   //------------------------------------------------------

   refresh_lcd++;

   if(refresh_lcd >= 500){

      refresh_lcd = 0;

      lcd_recovery();
   }

   //------------------------------------------------------
   // MODO OPERAÇÃO
   //------------------------------------------------------

   if(menu_nivel == 0){

      if(old_menu_nivel == menu_nivel &&
         old_contador == contador_espiras &&
         old_run == maquina_run &&
         old_falha == falha_inversor)
         return;

      old_menu_nivel = menu_nivel;

      old_contador = contador_espiras;

      old_run = maquina_run;

      old_falha = falha_inversor;

      //---------------------------------------------------
      // LINHA 1
      //---------------------------------------------------

      lcd_gotoxy(1,1);

      delay_ms(5);

      lcd_putc("ENROLAR ESPIRA");

      delay_ms(5);

      //---------------------------------------------------
      // LINHA 2
      //---------------------------------------------------

      lcd_gotoxy(1,2);

      delay_ms(5);

      //---------------------------------------------------
      // FALHA
      //---------------------------------------------------

      if(falha_inversor == 1){

         lcd_putc("FALHA INVERSOR");

         delay_ms(5);

         return;
      }

      //---------------------------------------------------
      // CONTADOR
      //---------------------------------------------------

      printf(lcd_putc,"%04Lu            ",
             contador_espiras);

      delay_ms(5);

      return;
   }

   //------------------------------------------------------
   // MENU 1
   //------------------------------------------------------

   if(menu_nivel == 1){

      if(old_menu_nivel == menu_nivel &&
         old_bobina_atual == bobina_atual)
         return;

      old_menu_nivel = menu_nivel;

      old_bobina_atual = bobina_atual;

      lcd_gotoxy(1,1);

      delay_ms(5);

      lcd_putc("BOBINA         ");

      delay_ms(5);

      lcd_gotoxy(1,2);

      delay_ms(5);

      printf(lcd_putc,"%u               ",
             bobina_atual);

      delay_ms(5);

      return;
   }
}

//---------------------------------------------------------
// MAIN
//---------------------------------------------------------

void main(){

   hardware_init();

   receitas_init();

   lcd_startup();

   while(TRUE){

      scan_keyboard();

      task_menu();

      task_start();

      task_encoder();

      task_display();

      delay_ms(10);
   }
}
