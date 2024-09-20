# Знакомство с Markdown и GitHub

## Кольцевой буфер

**Кольцевой буфер**, или **циклический буфер** (англ. *ring-buffer*) — это структура данных, использующая единственный буфер фиксированного размера таким образом, как будто бы после последнего элемента сразу же снова идет первый. Такая структура легко предоставляет возможность буферизации потоков данных.

~~Я зачёркунатая строка, я тут ни при чём!~~

### Принцип работы

Анимация, демонстрирующая принцип работы кольцевого буфера:  
![Ring buffer](Circular_Buffer_Animation.gif)

### Применение

Часто кольцевой буфер применяется при программировании микроконтроллеров, например:
- STM32
- Arduino
- ESP32

### Реализация

Пример реализации кольцевого буфера на языке c:
[t1m013y/RingBuf-c](https://github.com/t1m013y/RingBuf-c)

Некоторые функции из этой библиотеки и отрывки из документации:
```c
typedef struct {
  bool _wInit;
  volatile bool _isLocked;
  
  char *buffer;
  
  size_t buffer_size;
  
  size_t tail_index;
  size_t elements_count;
} RingBuf_t;
```

> The ring buffer and type. Used to create the buffer and by library functions. Stores data about the buffer.  
The buffer has locked flag. It is true if any process is modifying buffer at the moment. If this flag is true, any operation that can modify the buffer won't be started and will return 0. Some functions ignore locked flag.

```c
int RingBuf_Init(RingBuf_t *buffer_h, size_t buffer_size)
{
  if (!(buffer_size > 0))
    return 0;
  
  buffer_h->buffer_size = buffer_size;
  
  buffer_h->tail_index = 0;
  buffer_h->elements_count = 0;
  
  buffer_h->buffer = (char*)malloc(buffer_h->buffer_size);
  if (buffer_h->buffer == NULL) {
    buffer_h->_wInit = false;
    return 0;
  }
  
  buffer_h->_isLocked = false;
  buffer_h->_wInit = true;
  return 1;
}
```

> Initializes the ring buffer.

```c
int RingBuf_Queue(RingBuf_t *buffer_h, const char data)
{
  return RingBuf__Queue(buffer_h, data, false);
}
```

> Adds one element to buffer. If buffer is already full, it will overwrite the oldest element of the buffer.

```c
int RingBuf_Dequeue(RingBuf_t *buffer_h, char *data)
{
  return RingBuf__Dequeue(buffer_h, data, false);
}
```

> Reads one (oldest) element from the buffer and removes this element. If data is a null pointer, the element from the buffer will be removed but it will not be saved.
