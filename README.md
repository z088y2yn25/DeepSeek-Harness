零基础易懂：自定义函数实现数组逆置（附完整代码详解）
在编程学习中，数组是最基础、最常用的数据结构，而数组逆置是入门必练的经典案例。简单来说，数组逆置就是把数组中的元素顺序完全颠倒，比如原数组是[1,2,3,4,5]，逆置后变为[5,4,3,2,1]。
很多新手初学数组时，会直接套用语言自带的翻转函数，但真正的编程核心思维，是自己定义函数实现功能。手动编写自定义函数完成数组逆置，能帮我们彻底吃透数组下标操作、函数传参、循环逻辑等核心知识点，告别只会调用工具的“代码搬运”思维。
今天本文将用最通俗的语言、清晰的逻辑、完整的代码示例，手把手教大家通过自定义函数实现数组逆置，同时对比输出原数组与翻转后的数组，全程无晦涩术语，零基础也能轻松看懂。
一、先搞懂核心原理：数组逆置的逻辑本质
在写代码之前，我们先理清底层逻辑，不用死记硬背公式。数组的所有元素都依靠下标（索引）定位，下标从0开始依次递增。例如数组 arr = [10,20,30,40,50]，下标对应关系为：0→10、1→20、2→30、3→40、4→50。
数组逆置的核心思路非常简单：首尾对称交换元素，具体步骤如下：
1. 定义两个标记，一个指向数组头部（起始下标0），一个指向数组尾部（最后一个元素下标）；
2. 交换头部下标和尾部下标对应的元素值；
3. 头部标记向后移动一位，尾部标记向前移动一位；
4. 重复交换操作，直到两个标记相遇（所有元素交换完成）。
整个过程无需额外创建新数组，直接在原数组上修改，节省内存空间，也是编程中最推荐的原地逆置算法。需要注意的是，无论数组长度是奇数还是偶数，该逻辑都完全适用：奇数长度数组的中间元素无需交换，自动保留原位；偶数长度数组所有元素两两交换即可。
二、为什么要用自定义函数实现？
很多编程语言自带数组翻转方法，比如Python的reverse()、C++的reverse()函数，但我们坚持用自定义函数实现，有三大核心意义：
1. 吃透底层逻辑：系统函数是封装好的黑盒子，自定义函数需要自己编写循环、交换逻辑，能彻底理解逆置的实现原理；
2. 提升代码复用性：自定义函数封装完成后，后续任何数组需要逆置，直接调用函数即可，无需重复写代码，符合编程“高复用、低冗余”原则；
3. 适配自定义需求：系统函数功能固定，自定义函数可以灵活修改，比如实现部分数组逆置、逆置后输出格式化结果等个性化功能。
三、完整实战代码（以C语言为例，通用性强）
下面以入门最经典的C语言为例，编写完整可运行代码。代码核心分为三部分：自定义逆置函数、原数组输出、逆置后数组输出，逻辑清晰、注释详细，可直接复制运行。
#include <stdio.h>

// 自定义数组逆置函数
void reverseArray(int arr[], int len)
{
    // 定义首尾下标，循环交换元素
    int left = 0;         // 左指针：数组起始位置
    int right = len - 1;  // 右指针：数组末尾位置
    int temp;             // 临时变量，用于元素交换

    // 首尾指针未相遇时，持续交换
    while (left < right)
    {
        // 首尾元素交换
        temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        // 指针移动，缩小交换范围
        left++;
        right--;
    }
}

// 自定义数组打印函数（封装输出逻辑，简化代码）
void printArray(int arr[], int len)
{
    for (int i = 0; i < len; i++)
    {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main()
{
    // 定义原始数组
    int arr[] = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    // 计算数组长度
    int length = sizeof(arr) / sizeof(arr[0]);

    // 输出原数组
    printf("原数组元素：");
    printArray(arr, length);

    // 调用自定义函数实现数组逆置
    reverseArray(arr, length);

    // 输出翻转后的数组
    printf("逆置后数组：");
    printArray(arr, length);

    return 0;
}
四、代码逐行深度解析，彻底看懂每一步
1. 自定义逆置函数 reverseArray
函数参数包含两个核心内容：int arr[] 接收待逆置的数组，int len 接收数组长度，无返回值，直接对原数组进行修改。
通过 left 和 right 双指针实现对称交换：循环条件 left < right 是关键，当两个指针重合或交叉时，说明所有可交换的元素已经完成翻转，终止循环，避免重复交换导致数组还原。
临时变量 temp 是元素交换的必备载体，通过中转赋值，完成两个下标元素的数值互换。
2. 自定义打印函数 printArray
为了避免重复编写循环输出代码，我们单独封装打印函数，无论是输出原数组还是逆置后的数组，只需调用该函数即可，让代码更简洁、规整。
3. 主函数执行逻辑
首先定义初始数组，通过 sizeof 自动计算数组长度，无需手动赋值，适配任意长度数组；随后先打印原始数组，再调用逆置函数处理数组，最后打印翻转结果，流程清晰、结果对比直观。
五、程序运行结果展示
运行上述代码后，控制台会精准输出两组对比数据，清晰呈现逆置效果：
原数组元素：1 2 3 4 5 6 7 8 9
逆置后数组：9 8 7 6 5 4 3 2 1
大家可以自行修改数组元素、数组长度，测试奇数、偶数、无序数字等不同场景，代码均可正常运行。
六、拓展思考：新手常见误区解答
误区1：循环条件写成 left <= right
如果数组长度为奇数，当 left == right 时，指针指向数组中间元素，此时无需交换。若继续执行循环，会自己和自己交换，无效果但浪费运算资源，属于冗余代码。
误区2：不封装函数，直接在主函数写循环
虽然可以实现效果，但代码复用性极差。如果项目中有多个数组需要逆置，需要重复写多遍循环代码，代码臃肿且不利于维护，不符合规范化编程思维。
误区3：额外创建新数组存储逆置结果
该方法可行，但会占用额外内存。而本文的原地交换算法，仅用一个临时变量，空间复杂度更低，是更优质的编程解法。
七、总结
数组逆置看起来是简单的基础案例，却完美涵盖了自定义函数封装、数组下标操作、双指针算法、循环逻辑优化四大编程核心知识点。
通过自定义函数实现数组逆置，核心不是实现“翻转效果”，而是学会拆解问题、封装功能、优化代码的编程思维。掌握这个逻辑后，大家不仅能轻松应对各类数组翻转题型，还能为后续学习排序算法、字符串操作、数据结构打下坚实的基础。
后续练习中，大家可以尝试拓展功能，比如实现输入自定义数组、逆置后去除重复元素、字符串数组逆置等进阶操作，逐步提升编程能力。# DeepSeek-Harness
DeepSeek Harness


https://www.v-tianjin.com/user/8601/questions
https://www.v-tianjin.com/user/2240/questions
https://www.v-tianjin.com/user/4851/favorites
https://www.v-tianjin.com/user/8130/favorites
https://www.v-tianjin.com/user/7295/follows
https://www.v-tianjin.com/user/2209/favorites
https://www.v-tianjin.com/user/4669/favorites
https://www.v-tianjin.com/user/5688/follows
https://www.v-tianjin.com/user/8796/follows
https://www.v-tianjin.com/user/6075/favorites
https://www.v-tianjin.com/user/8796/questions
https://www.v-tianjin.com/user/2796/follows
https://www.v-tianjin.com/user/5673/follows
https://www.v-tianjin.com/user/2975/favorites
https://www.v-tianjin.com/user/3982/follows
https://www.v-tianjin.com/user/8728/questions
https://www.v-tianjin.com/user/7102/questions
https://www.v-tianjin.com/user/2975/questions
https://www.v-tianjin.com/user/9737/favorites
https://www.v-tianjin.com/user/2240/favorites
https://www.v-tianjin.com/user/7316/favorites
https://www.v-tianjin.com/user/6231/favorites
https://www.v-tianjin.com/user/8755/questions
https://www.v-tianjin.com/user/2188/follows
https://www.v-tianjin.com/user/2429/favorites
https://www.v-tianjin.com/user/2792/favorites
https://www.v-tianjin.com/user/4487/questions
https://www.v-tianjin.com/user/1888/questions
https://www.v-tianjin.com/user/8318/follows
https://www.v-tianjin.com/user/2796/favorites
https://www.v-tianjin.com/user/2252/favorites
https://www.v-tianjin.com/user/4442/favorites
https://www.v-tianjin.com/user/7958/follows
https://www.v-tianjin.com/user/9976/favorites
https://www.v-tianjin.com/user/7679/follows
https://www.v-tianjin.com/user/9348/follows
https://www.v-tianjin.com/user/4042/favorites
https://www.v-tianjin.com/user/1216/favorites
https://www.v-tianjin.com/user/6874/follows
https://www.v-tianjin.com/user/7574/follows
https://www.v-tianjin.com/user/2667/follows
https://www.v-tianjin.com/user/6874/favorites
https://www.v-tianjin.com/user/9737/follows
https://www.v-tianjin.com/user/5947/questions
https://www.v-tianjin.com/user/8500/follows
https://www.v-tianjin.com/user/7302/questions
https://www.v-tianjin.com/user/5688/questions
https://www.v-tianjin.com/user/7947/favorites
https://www.v-tianjin.com/user/3445/favorites
https://www.v-tianjin.com/user/8507/favorites
https://www.v-tianjin.com/user/7823/follows
https://www.v-tianjin.com/user/3644/follows
https://www.v-tianjin.com/user/5688/favorites
https://www.v-tianjin.com/user/7958/favorites
https://www.v-tianjin.com/user/8507/follows
https://www.v-tianjin.com/user/8130/follows
https://www.v-tianjin.com/user/6874/questions
https://www.v-tianjin.com/user/5673/questions
https://www.v-tianjin.com/user/8126/follows
https://www.v-tianjin.com/user/1888/favorites
https://www.v-tianjin.com/user/8500/favorites
https://www.v-tianjin.com/user/6231/questions
https://www.v-tianjin.com/user/8728/favorites
https://www.v-tianjin.com/user/8728/follows
https://www.v-tianjin.com/user/8518/questions
https://www.v-tianjin.com/user/7481/questions
https://www.v-tianjin.com/user/4475/questions
https://www.v-tianjin.com/user/9325/favorites
https://www.v-tianjin.com/user/8318/favorites
https://www.v-tianjin.com/user/2792/questions
https://www.v-tianjin.com/user/3445/questions
https://www.v-tianjin.com/user/1216/follows
https://www.v-tianjin.com/user/8545/favorites
https://www.v-tianjin.com/user/4559/follows
https://www.v-tianjin.com/user/7574/favorites
https://www.v-tianjin.com/user/7243/follows
https://www.v-tianjin.com/user/2223/follows
https://www.v-tianjin.com/user/8507/questions
https://www.v-tianjin.com/user/8500/questions
https://www.v-tianjin.com/user/8126/favorites
https://www.v-tianjin.com/user/6075/follows
https://www.v-tianjin.com/user/2403/questions
https://www.v-tianjin.com/user/7309/questions
https://www.v-tianjin.com/user/5796/follows
https://www.v-tianjin.com/user/7309/follows
https://www.v-tianjin.com/user/7771/favorites
https://www.v-tianjin.com/user/3230/favorites
https://www.v-tianjin.com/user/8562/questions
https://www.v-tianjin.com/user/7296/follows
https://www.v-tianjin.com/user/7296/favorites
https://www.v-tianjin.com/user/7481/favorites
https://www.v-tianjin.com/user/3086/follows
https://www.v-tianjin.com/user/7309/favorites
https://www.v-tianjin.com/user/3086/questions
https://www.v-tianjin.com/user/9542/follows
https://www.v-tianjin.com/user/2195/favorites
https://www.v-tianjin.com/user/7302/follows
https://www.v-tianjin.com/user/5796/questions
https://www.v-tianjin.com/user/5457/follows
https://www.v-tianjin.com/user/4851/follows
https://www.v-tianjin.com/user/32601/questions
https://www.v-tianjin.com/user/2796/questions
https://www.v-tianjin.com/user/9416/favorites
https://www.v-tianjin.com/user/20071/follows
https://www.v-tianjin.com/user/2188/favorites
https://www.v-tianjin.com/user/1682/questions
https://www.v-tianjin.com/user/4956/follows
https://www.v-tianjin.com/user/8518/follows
https://www.v-tianjin.com/user/7683/questions
https://www.v-tianjin.com/user/4669/questions
https://www.v-tianjin.com/user/8545/follows
https://www.v-tianjin.com/user/2851/questions
https://www.v-tianjin.com/user/9542/questions
https://www.v-tianjin.com/user/9325/questions
https://www.v-tianjin.com/user/2252/questions
https://www.v-tianjin.com/user/1054/favorites
https://www.v-tianjin.com/user/1682/favorites
https://www.v-tianjin.com/user/44401/follows
https://www.v-tianjin.com/user/7907/questions
https://www.v-tianjin.com/user/7683/follows
https://www.v-tianjin.com/user/3230/questions
https://www.v-tianjin.com/user/6075/questions
https://www.v-tianjin.com/user/7823/favorites
https://www.v-tianjin.com/user/3086/favorites
https://www.v-tianjin.com/user/5236/favorites
https://www.v-tianjin.com/user/1888/follows
https://www.v-tianjin.com/user/5796/favorites
https://www.v-tianjin.com/user/7302/favorites
https://www.v-tianjin.com/user/5236/follows
https://www.v-tianjin.com/user/8545/questions
https://www.v-tianjin.com/user/8146/favorites
https://www.v-tianjin.com/user/7243/questions
https://www.v-tianjin.com/user/7295/favorites
https://www.v-tianjin.com/user/4487/follows
https://www.v-tianjin.com/user/3445/follows
https://www.v-tianjin.com/user/7243/favorites
https://www.v-tianjin.com/user/8859/favorites
https://www.v-tianjin.com/user/7683/favorites
https://www.v-tianjin.com/user/7907/follows
https://www.v-tianjin.com/user/2975/follows
https://www.v-tianjin.com/user/4042/follows
https://www.v-tianjin.com/user/4375/questions
https://www.v-tianjin.com/user/9250/favorites
https://www.v-tianjin.com/user/6943/favorites
https://www.v-tianjin.com/user/9542/favorites
https://www.v-tianjin.com/user/7771/questions
https://www.v-tianjin.com/user/4442/questions
https://www.v-tianjin.com/user/2851/favorites
https://www.v-tianjin.com/user/4559/favorites
https://www.v-tianjin.com/user/32601/favorites
https://www.v-tianjin.com/user/8859/questions
https://www.v-tianjin.com/user/5947/favorites
https://www.v-tianjin.com/user/7823/questions
https://www.v-tianjin.com/user/2851/follows
https://www.v-tianjin.com/user/1523/follows
https://www.v-tianjin.com/user/4042/questions
https://www.v-tianjin.com/user/8755/follows
https://www.v-tianjin.com/user/2209/follows
https://www.v-tianjin.com/user/44401/favorites
https://www.v-tianjin.com/user/9737/questions
https://www.v-tianjin.com/user/5539/favorites
https://www.v-tianjin.com/user/7947/follows
https://www.v-tianjin.com/user/8146/questions
https://www.v-tianjin.com/user/4327/follows
https://www.v-tianjin.com/user/4851/questions
https://www.v-tianjin.com/user/2223/favorites
https://www.v-tianjin.com/user/8318/questions
https://www.v-tianjin.com/user/7907/favorites
https://www.v-tianjin.com/user/32601/follows
https://www.v-tianjin.com/user/2195/follows
https://www.v-tianjin.com/user/9416/questions
https://www.v-tianjin.com/user/9250/questions
https://www.v-tianjin.com/user/2209/questions
https://www.v-tianjin.com/user/7689/questions
https://www.v-tianjin.com/user/20071/questions
https://www.v-tianjin.com/user/7102/follows
https://www.v-tianjin.com/user/2252/follows
https://www.v-tianjin.com/user/7316/questions
https://www.v-tianjin.com/user/7958/questions
https://www.v-tianjin.com/user/7789/favorites
https://www.v-tianjin.com/user/2403/follows
https://www.v-tianjin.com/user/2792/follows
https://www.v-tianjin.com/user/3644/questions
https://www.v-tianjin.com/user/9348/questions
https://www.v-tianjin.com/user/2429/follows
https://www.v-tianjin.com/user/5539/follows
https://www.v-tianjin.com/user/6943/questions
https://www.v-tianjin.com/user/8146/follows
https://www.v-tianjin.com/user/4021/follows
https://www.v-tianjin.com/user/1523/favorites
https://www.v-tianjin.com/user/4559/questions
https://www.v-tianjin.com/user/8518/favorites
https://www.v-tianjin.com/user/8859/follows
https://www.v-tianjin.com/user/4442/follows
https://www.v-tianjin.com/user/2223/questions
https://www.v-tianjin.com/user/9976/questions
https://www.v-tianjin.com/user/7102/favorites
https://www.v-tianjin.com/user/7771/follows
https://www.v-tianjin.com/user/8601/favorites
https://www.v-tianjin.com/user/7679/questions
https://www.v-tianjin.com/user/2429/questions
https://www.v-tianjin.com/user/8796/favorites
https://www.v-tianjin.com/user/8474/follows
https://www.v-tianjin.com/user/5673/favorites
https://www.v-tianjin.com/user/4375/favorites
https://www.v-tianjin.com/user/7789/follows
https://www.v-tianjin.com/user/1523/questions
https://www.v-tianjin.com/user/1054/questions
https://www.v-tianjin.com/user/3230/follows
https://www.v-tianjin.com/user/4021/favorites
https://www.v-tianjin.com/user/7789/questions
https://www.v-tianjin.com/user/2240/follows
https://www.v-tianjin.com/user/8126/questions
https://www.v-tianjin.com/user/9976/follows
https://www.v-tianjin.com/user/8562/favorites
https://www.v-tianjin.com/user/7679/favorites
https://www.v-tianjin.com/user/1216/questions
https://www.v-tianjin.com/user/8601/follows
https://www.v-tianjin.com/user/4487/favorites
https://www.v-tianjin.com/user/2667/questions
https://www.v-tianjin.com/user/3982/questions
https://www.v-tianjin.com/user/4021/questions
https://www.v-tianjin.com/user/4956/favorites
https://www.v-tianjin.com/user/5457/favorites
https://www.v-tianjin.com/user/4327/favorites
https://www.v-tianjin.com/user/44401/questions
https://www.v-tianjin.com/user/6943/follows
https://www.v-tianjin.com/user/7947/questions
https://www.v-tianjin.com/user/1054/follows
https://www.v-tianjin.com/user/8474/favorites
https://www.v-tianjin.com/user/9348/favorites
https://www.v-tianjin.com/user/2195/questions
https://www.v-tianjin.com/user/8474/questions
https://www.v-tianjin.com/user/2188/questions
https://www.v-tianjin.com/user/1682/follows
https://www.v-tianjin.com/user/4475/follows
https://www.v-tianjin.com/user/6231/follows
https://www.v-tianjin.com/user/9250/follows
https://www.v-tianjin.com/user/5947/follows
https://www.v-tianjin.com/user/5236/questions
https://www.v-tianjin.com/user/4669/follows
https://www.v-tianjin.com/user/8130/questions
https://www.v-tianjin.com/user/2403/favorites
https://www.v-tianjin.com/user/7295/questions
https://www.v-tianjin.com/user/7574/questions
https://www.v-tianjin.com/user/3982/favorites
https://www.v-tianjin.com/user/7296/questions
https://www.v-tianjin.com/user/4375/follows
https://www.v-tianjin.com/user/8755/favorites
https://www.v-tianjin.com/user/4475/favorites
https://www.v-tianjin.com/user/5539/questions
https://www.v-tianjin.com/user/7689/follows
https://www.v-tianjin.com/user/7316/follows
https://www.v-tianjin.com/user/20071/favorites
https://www.v-tianjin.com/user/5457/questions
https://www.v-tianjin.com/user/7481/follows
https://www.v-tianjin.com/user/8562/follows
https://www.v-tianjin.com/user/9325/follows
https://www.v-tianjin.com/user/4956/questions
https://www.v-tianjin.com/user/2667/favorites
https://www.v-tianjin.com/user/9416/follows
https://www.v-tianjin.com/user/7689/favorites
https://www.v-tianjin.com/user/4327/questions
https://www.v-tianjin.com/user/3644/favorites
https://shequ.docin.com/app/shequ/themeview?tid=3179581
https://shequ.docin.com/app/shequ/themeview?tid=3179743
https://shequ.docin.com/app/shequ/themeview?tid=3179741
https://shequ.docin.com/app/shequ/themeview?tid=3179740
https://shequ.docin.com/app/shequ/themeview?tid=3179739
https://shequ.docin.com/app/shequ/themeview?tid=3179738
https://shequ.docin.com/app/shequ/themeview?tid=3179737
https://shequ.docin.com/app/shequ/themeview?tid=3179736
https://shequ.docin.com/app/shequ/themeview?tid=3179733
https://shequ.docin.com/app/shequ/themeview?tid=3179733
https://shequ.docin.com/app/shequ/themeview?tid=3179732
https://shequ.docin.com/app/shequ/themeview?tid=3179691
https://shequ.docin.com/app/shequ/themeview?tid=3179689
https://shequ.docin.com/app/shequ/themeview?tid=3179688
https://shequ.docin.com/app/shequ/themeview?tid=3179687
https://shequ.docin.com/app/shequ/themeview?tid=3179686
https://shequ.docin.com/app/shequ/themeview?tid=3179685
https://shequ.docin.com/app/shequ/themeview?tid=3179684
https://shequ.docin.com/app/shequ/themeview?tid=3179683
https://shequ.docin.com/app/shequ/themeview?tid=3179681
https://shequ.docin.com/app/shequ/themeview?tid=3179680
https://shequ.docin.com/app/shequ/themeview?tid=3179679
https://shequ.docin.com/app/shequ/themeview?tid=3179678
https://shequ.docin.com/app/shequ/themeview?tid=3179676
https://shequ.docin.com/app/shequ/themeview?tid=3179675
https://shequ.docin.com/app/shequ/themeview?tid=3179674
https://shequ.docin.com/app/shequ/themeview?tid=3179673
https://shequ.docin.com/app/shequ/themeview?tid=3179672
https://shequ.docin.com/app/shequ/themeview?tid=3179670
https://shequ.docin.com/app/shequ/themeview?tid=3179666
https://shequ.docin.com/app/shequ/themeview?tid=3179664
https://shequ.docin.com/app/shequ/themeview?tid=3179663
https://shequ.docin.com/app/shequ/themeview?tid=3179662
https://shequ.docin.com/app/shequ/themeview?tid=3179660
https://shequ.docin.com/app/shequ/themeview?tid=3179659
https://shequ.docin.com/app/shequ/themeview?tid=3179658
https://shequ.docin.com/app/shequ/themeview?tid=3179656
https://shequ.docin.com/app/shequ/themeview?tid=3179654
https://shequ.docin.com/app/shequ/themeview?tid=3179652
https://shequ.docin.com/app/shequ/themeview?tid=3179650
https://shequ.docin.com/app/shequ/themeview?tid=3179649
https://shequ.docin.com/app/shequ/themeview?tid=3179647
https://shequ.docin.com/app/shequ/themeview?tid=3179646
https://shequ.docin.com/app/shequ/themeview?tid=3179645
https://shequ.docin.com/app/shequ/themeview?tid=3179644
https://shequ.docin.com/app/shequ/themeview?tid=3179636
https://shequ.docin.com/app/shequ/themeview?tid=3179635
https://shequ.docin.com/app/shequ/themeview?tid=3179634
https://shequ.docin.com/app/shequ/themeview?tid=3179633
https://shequ.docin.com/app/shequ/themeview?tid=3179631
https://shequ.docin.com/app/shequ/themeview?tid=3179630
https://shequ.docin.com/app/shequ/themeview?tid=3179628
https://shequ.docin.com/app/shequ/themeview?tid=3179627
https://shequ.docin.com/app/shequ/themeview?tid=3179626
https://shequ.docin.com/app/shequ/themeview?tid=3179625
https://shequ.docin.com/app/shequ/themeview?tid=3179581
https://shequ.docin.com/app/shequ/themeview?tid=3179580
https://shequ.docin.com/app/shequ/themeview?tid=3179579
https://shequ.docin.com/app/shequ/themeview?tid=3179578
https://shequ.docin.com/app/shequ/themeview?tid=3179575
https://shequ.docin.com/app/shequ/themeview?tid=3179574
https://shequ.docin.com/app/shequ/themeview?tid=3179573
https://shequ.docin.com/app/shequ/themeview?tid=3179572
https://shequ.docin.com/app/shequ/themeview?tid=3179571
https://shequ.docin.com/app/shequ/themeview?tid=3178879



https://shequ.docin.com/app/shequ/themeview?tid=3179907
https://shequ.docin.com/app/shequ/themeview?tid=3179910
https://shequ.docin.com/app/shequ/themeview?tid=3179911
https://shequ.docin.com/app/shequ/themeview?tid=3179912
https://shequ.docin.com/app/shequ/themeview?tid=3179914
https://shequ.docin.com/app/shequ/themeview?tid=3179918
https://shequ.docin.com/app/shequ/themeview?tid=3179921
https://shequ.docin.com/app/shequ/themeview?tid=3179923
https://shequ.docin.com/app/shequ/themeview?tid=3179926
https://shequ.docin.com/app/shequ/themeview?tid=3179930
https://shequ.docin.com/app/shequ/themeview?tid=3179932
https://shequ.docin.com/app/shequ/themeview?tid=3179935
https://shequ.docin.com/app/shequ/themeview?tid=3179936
https://shequ.docin.com/app/shequ/themeview?tid=3179939
https://shequ.docin.com/app/shequ/themeview?tid=3179941
https://shequ.docin.com/app/shequ/themeview?tid=3179943
https://shequ.docin.com/app/shequ/themeview?tid=3179945
https://shequ.docin.com/app/shequ/themeview?tid=3179947
https://shequ.docin.com/app/shequ/themeview?tid=3179950
https://shequ.docin.com/app/shequ/themeview?tid=3179956
https://shequ.docin.com/app/shequ/themeview?tid=3179957
https://shequ.docin.com/app/shequ/themeview?tid=3179959
https://shequ.docin.com/app/shequ/themeview?tid=3179962
https://shequ.docin.com/app/shequ/themeview?tid=3179964
https://shequ.docin.com/app/shequ/themeview?tid=3179967
https://shequ.docin.com/app/shequ/themeview?tid=3179968
https://shequ.docin.com/app/shequ/themeview?tid=3179969
https://shequ.docin.com/app/shequ/themeview?tid=3179971
https://shequ.docin.com/app/shequ/themeview?tid=3179973
https://shequ.docin.com/app/shequ/themeview?tid=3179975
https://shequ.docin.com/app/shequ/themeview?tid=3179976
https://shequ.docin.com/app/shequ/themeview?tid=3179979
https://shequ.docin.com/app/shequ/themeview?tid=3179981
https://learnku.com/articles/93721
https://learnku.com/articles/93722
https://learnku.com/articles/93723
https://learnku.com/articles/93726
https://learnku.com/articles/93727
https://learnku.com/articles/93728
https://learnku.com/articles/93729
https://learnku.com/articles/93731
https://learnku.com/articles/93961
https://learnku.com/articles/93962
https://learnku.com/articles/93963
https://learnku.com/articles/93964
https://learnku.com/articles/93965
https://learnku.com/articles/93966
https://learnku.com/articles/93967
https://learnku.com/articles/93968
https://learnku.com/articles/93969
https://learnku.com/articles/93970
https://learnku.com/articles/93971
https://learnku.com/articles/93972
https://learnku.com/articles/93973
https://learnku.com/articles/93974
https://learnku.com/articles/93975

