---
title: Включение USART на STM32F0
layout: post
---
USART на F0 включается очень просто!

```
#define BAUDRATE	19200
#define F_CPU		48000000

void InitUSART1() {
	RCC->CR	|=	RCC_CR_HSION;			// Включить генератор HSE.
	while (!(RCC->CR & RCC_CR_HSIRDY)) {};		// Ожидание готовности HSE.
	RCC->CFGR |= RCC_CFGR_SW_HSI;			// Выбираем источником тактовой частоты SYSCLK генератор HSI
	RCC->CR &= ~RCC_CR_HSEON;			// Отключаем генератор MSI.

	GPIOA->MODER	|= GPIO_MODER_MODER9_1;		// Включаем порт на доп. ф-ции
	GPIOA->OSPEEDR	|= GPIO_OSPEEDER_OSPEEDR9;	// PA9 (TX) - High speed
	GPIOA->OTYPER &= ~GPIO_OTYPER_OT_9;		// PA9 - выход push-pull
	GPIOA->PUPDR &= ~(GPIO_PUPDR_PUPDR9);		// PA9 - без подтяжки
	GPIOA->AFR[1]	|= 0x0111;			// Врубаем доп. ф-ции на порте
	RCC->APB2ENR	|= RCC_APB2ENR_USART1EN;	// Включаем USART порт

	USART1->CR1 &= ~USART_CR1_M;			// 8 бит данных
	USART1->CR2 &= ~USART_CR2_STOP;			// 1 стоп-бит
	USART1->BRR = (F_CPU+BAUDRATE/2)/BAUDRATE;	// Выставим битрейт для порта
	USART1->CR1  |= USART_CR1_UE | USART_CR1_TE | USART_CR1_RE;
}
```

В общем-то все стандартно, но главное ничего не забыть в этом коде. Можно иметь ввиду, что
режим работы порта 8N1 это стандартный режим и не обязательно его выставлять явно.