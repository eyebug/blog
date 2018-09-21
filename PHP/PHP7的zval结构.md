#PHP5与PHP7的ZVAL结构


##PHP5 中的 zval
```c
// 1. zval
typedef struct _zval_struct {
    zvalue_value value;
    zend_uint refcount__gc;
    zend_uchar type;
    zend_uchar is_ref__gc;
} zval;

// 2. zvalue_value
typedef union _zvalue_value {
    long lval;     // 用于 bool 类型、整型和资源类型
    double dval;    // 用于浮点类型
    struct {     // 用于字符串
        char *val;
        int len;
    } str;
    HashTable *ht;    // 用于数组
    zend_object_value obj;  // 用于对象
    zend_ast *ast;    // 用于常量表达式(PHP5.6 才有)
} zvalue_value;

// 3. zend_object_value
typedef struct _zend_object_value {
    zend_object_handle handle;
    const zend_object_handlers *handlers;
} zend_object_value;

// 4. zend_object_handle
typedef unsigned int zend_object_handle;
```
多数文章，在提到PHP5 变量结构体的时候，都提到：sizeof(zval) == 24, sizeof(zvalue\_value) == 16，实际上这个论述并不准确，在 CPU 为 64bit 时，这个结果是正确的。但当 CPU 为32bit 时： sizeof(zval) == 16, sizeof(zvalue_value) == 8，主要因为 CPU 为 64bit 时，指针占用8个字节，而 32bit时，指针为4个字节。

##PHP 7 中的 zval
```c
// 1. zval
struct _zval_struct {
    zend_value        value;            /* value */
    union {
        struct {
            ZEND_ENDIAN_LOHI_4(
                zend_uchar    type,            /* active type */
                zend_uchar    type_flags,
                zend_uchar    const_flags,
                zend_uchar    reserved)        /* call info for EX(This) */
        } v;
        uint32_t type_info;
    } u1;
    union {
        uint32_t     next;                 /* hash collision chain */
        uint32_t     cache_slot;           /* literal cache slot */
        uint32_t     lineno;               /* line number (for ast nodes) */
        uint32_t     num_args;             /* arguments number for EX(This) */
        uint32_t     fe_pos;               /* foreach position */
        uint32_t     fe_iter_idx;          /* foreach iterator index */
        uint32_t     access_flags;         /* class constant access flags */
        uint32_t     property_guard;       /* single property guard */
    } u2;
};

// 2. zend_value
typedef union _zend_value {
    zend_long         lval;                /* long value */
    double            dval;                /* double value */
    zend_refcounted  *counted;
    zend_string      *str;
    zend_array       *arr;
    zend_object      *obj;
    zend_resource    *res;
    zend_reference   *ref;
    zend_ast_ref     *ast;
    zval             *zv;
    void             *ptr;
    zend_class_entry *ce;
    zend_function    *func;
    struct {
        uint32_t w1;
        uint32_t w2;
    } ww;
} zend_value;
```
PHP7 的看似很多，但其实更简单了，不论 CPU 是32bit 还是 64bit，sizeof(zval) 永远都是等于 16。主要看 zend\_value 中的 ww，是两个 uint32\_t，这个永远是 8 个字节，所以 sizeof(zend\_value) == 8，因此 sizeof(zval) == 16。

