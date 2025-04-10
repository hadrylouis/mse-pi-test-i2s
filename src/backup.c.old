#include <zephyr/kernel.h>
#include <zephyr/device.h>
#include <zephyr/drivers/i2s.h>
#include <zephyr/drivers/gpio.h>

#define SAMPLE_BIT_WIDTH     32
#define NUMBER_OF_CHANNELS   1
#define SAMPLE_FREQUENCY     12000
#define BYTES_PER_SAMPLE     4
#define SAMPLES_PER_BLOCK    1200
#define BLOCK_SIZE           (BYTES_PER_SAMPLE * NUMBER_OF_CHANNELS * SAMPLES_PER_BLOCK)
#define BLOCK_COUNT          4

K_MEM_SLAB_DEFINE_STATIC(i2s_slab, BLOCK_SIZE, BLOCK_COUNT, 4);

static const uint32_t 	store_block_size		= (10 * SAMPLES_PER_BLOCK * BYTES_PER_SAMPLE);
static int8_t 			store_block[2][(10 * SAMPLES_PER_BLOCK * BYTES_PER_SAMPLE) + 1];


#define I2S_NODE DT_NODELABEL(i2s20)
static const struct device *i2s_dev = DEVICE_DT_GET(I2S_NODE);

#define MIC_WAKE_NODE  DT_NODELABEL(mic_wake)
#define MIC_OE_NODE    DT_NODELABEL(mic_oe)
#define MIC_THSEL_NODE DT_NODELABEL(mic_thsel)

static const struct gpio_dt_spec mic_wake  = GPIO_DT_SPEC_GET(MIC_WAKE_NODE, gpios);
static const struct gpio_dt_spec mic_oe    = GPIO_DT_SPEC_GET(MIC_OE_NODE, gpios);
static const struct gpio_dt_spec mic_thsel = GPIO_DT_SPEC_GET(MIC_THSEL_NODE, gpios);

void main(void)
{
    
    // DEBUG
    //gpio_pin_configure_dt(&mic_thsel, GPIO_OUTPUT_HIGH);
    //gpio_pin_configure_dt(&mic_wake, GPIO_INPUT);
    gpio_pin_configure_dt(&mic_wake, GPIO_DISCONNECTED);
    gpio_pin_configure_dt(&mic_thsel, GPIO_DISCONNECTED); // disable THSEL (temporary)
    gpio_pin_configure_dt(&mic_oe, GPIO_OUTPUT_HIGH);

    printk("MIC_WAKE: %d, MIC_OE: %d, MIC_THSEL: %d\n",
        gpio_pin_get_dt(&mic_wake),
        gpio_pin_get_dt(&mic_oe),
        gpio_pin_get_dt(&mic_thsel));

    /*
    printk("I2S RX Example\n");

    gpio_pin_configure_dt(&mic_wake, GPIO_INPUT);
    gpio_pin_configure_dt(&mic_thsel, GPIO_OUTPUT_LOW);
    gpio_pin_configure_dt(&mic_oe, GPIO_OUTPUT_HIGH);
    
    

    printk("MIC_WAKE: %d, MIC_OE: %d, MIC_THSEL: %d\n",
           gpio_pin_get_dt(&mic_wake),
           gpio_pin_get_dt(&mic_oe),
           gpio_pin_get_dt(&mic_thsel));

    if (!device_is_ready(i2s_dev)) {
        printk("I2S device not ready\n");
        return;
    }

    struct i2s_config cfg = {
        .word_size       = SAMPLE_BIT_WIDTH,
        .channels        = NUMBER_OF_CHANNELS,
        .format          = I2S_FMT_DATA_FORMAT_I2S,
        .options         = I2S_OPT_FRAME_CLK_MASTER | I2S_OPT_BIT_CLK_MASTER,
        .frame_clk_freq  = SAMPLE_FREQUENCY,
        .block_size      = BLOCK_SIZE,
        .mem_slab        = &i2s_slab,
        .timeout         = 2000,
    };

    int ret = i2s_configure(i2s_dev, I2S_DIR_RX, &cfg);
    if (ret < 0) {
        printk("I2S config failed: %d\n", ret);
        return;
    }

    // Pré-allouer et fournir 2 buffers au driver (double-buffering)
    for (int i = 0; i < 2; i++) {
        void *block;
        ret = k_mem_slab_alloc(&i2s_slab, &block, K_NO_WAIT);
        printk("Alloc block @ %p (align 4: %d)\n", block, ((uintptr_t)block % 4 == 0));
        
        if (ret != 0) {
            printk("Failed to allocate RX block\n");
            return;
        }
        ret = i2s_read(i2s_dev, &block, NULL);
        if (ret != 0) {
            printk("Failed to queue initial buffer: %d\n", ret);
            return;
        }
    }

    i2s_trigger(i2s_dev, I2S_DIR_RX, I2S_TRIGGER_PREPARE);
    ret = i2s_trigger(i2s_dev, I2S_DIR_RX, I2S_TRIGGER_START);
    if (ret < 0) {
        printk("I2S trigger failed: %d\n", ret);
        return;
    }
    printk("trigger OK");

    while (1) {
        void *rx_block;
        size_t rx_size;

        ret = i2s_read(i2s_dev, &rx_block, &rx_size);
        if (ret == 0) {
            printk("Received %u bytes\n", rx_size);

            uint8_t *buf = (uint8_t *)rx_block;

            for (int i = 0; i < 4 * BYTES_PER_SAMPLE; i += BYTES_PER_SAMPLE) {
                int32_t sample = ((int32_t)buf[i] << 24) |
                                 ((int32_t)buf[i + 1] << 16) |
                                 ((int32_t)buf[i + 2] << 8);
                sample >>= 8; // mettre en 24 bits signé
                printk("Sample[%d] = %d\n", i / BYTES_PER_SAMPLE, sample);
            }

            k_mem_slab_free(&i2s_slab, &rx_block);
        } else {
            printk("Read error: %d\n", ret);
        }

        k_msleep(100);
    }*/

    if (!device_is_ready(i2s_dev)) {
        printk("I2S device not ready\n");
        return;
    }

    #if DT_NODE_HAS_STATUS(I2S_NODE, okay)
    {
        printk("I2S device is ready\n");
    }
    #else
    {
        printk("I2S device is not ready\n");
        return;
    }
    #endif

    struct i2s_config cfg = {
        .word_size      = SAMPLE_BIT_WIDTH,
        .channels       = NUMBER_OF_CHANNELS,
        .format         = I2S_FMT_DATA_FORMAT_I2S,
        .options        = I2S_OPT_FRAME_CLK_MASTER | I2S_OPT_BIT_CLK_MASTER,
        .frame_clk_freq = SAMPLE_FREQUENCY,
        .block_size     = BLOCK_SIZE,
        .mem_slab       = &i2s_slab,
        .timeout        = 2000,
    };

    int ret = i2s_configure(i2s_dev, I2S_DIR_BOTH, &cfg);
    if (ret != 0) {
        printk("I2S configure failed: %d\n", ret);
        return;
    }

    //i2s_trigger(i2s_dev, I2S_DIR_BOTH, I2S_TRIGGER_PREPARE);

    // Pré-allocation des blocs RX (double buffering)
    for (int i = 0; i < 2; i++) {
        void *block;
        ret = k_mem_slab_alloc(&i2s_slab, &block, K_NO_WAIT);
        if (ret != 0) {
            printk("Failed to allocate RX block %d\n", i);
            return;
        }
        memset(block, 0, BLOCK_SIZE); // Initialiser le bloc à zéro

        ret = i2s_write(i2s_dev, block, BLOCK_SIZE);
        if (ret != 0) {
            printk("Failed to write initial buffer: %d\n", ret);
            return;
        }
    }

    ret = i2s_trigger(i2s_dev, I2S_DIR_BOTH, I2S_TRIGGER_START);
    if (ret != 0) {
        printk("Failed to start I2S: %d\n", ret);
        return;
    }

    printk("I2S started successfully\n");

    while (1) {
        void *rx_block;
        size_t rx_size = 0;

        ret = i2s_read(i2s_dev, &rx_block, &rx_size);
        if (ret == 0) {
            printk("RX received %u bytes\n", rx_size);
            //k_mem_slab_free(&i2s_slab, &rx_block);
        } else {
            printk("Read error: %d\n", ret);
        }

        k_sleep(K_MSEC(10));
    }
    
}
