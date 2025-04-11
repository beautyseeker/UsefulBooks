# 电路(Circuits)
## 组合逻辑(Combineational Logic)
### 基本门电路(Basic gates)
### 选择器(Multiplexer)
### 算术电路(Arthmetric Circuits)
- 有符号加法溢出(Signed addition overflow)
  ```Verilog
  //有符号数做加法，符号位溢出仅可能发生在符号位同号时，是否溢出取决于和的符号位以及符号进位是否一致，一致则不溢出，不一致则溢出。
  module top_module (
      input [7:0] a,
      input [7:0] b,
      output [7:0] s,
      output overflow
  );
      wire sum_carry;
      assign {sum_carry, s} = a + b;
      assign overflow = (a[7] ~^ b[7]) && (sum_carry ^ s[7]);

  endmodule
  ```

- 100位全加器(100-bit binary adder)
    ```Verilog
    // 加法进位，一步到位
    module top_module( 
    input [99:0] a, b,
    input cin,
    output cout,
    output [99:0] sum );
	
      assign {cout, sum} = a + b + cin;
  endmodule
    ```

- 4位BCD码加法器(Bcdadd4)
  ```Verilog
  //注意这里是BCD码的加法器，不过是4位BCD码(总计16位)的加法，直接调用给出的bcd_fadd模块串接起来，对应好位置正确传参就行。
  module top_module ( 
    input [15:0] a, b,
    input cin,
    output cout,
    output [15:0] sum
  );
  wire [3:0] cout_vec;
    genvar i;
    generate
        for(i = 0; i < 4; i = i + 1) begin: bcd_fadd_loop
            bcd_fadd call (
                a[4*i+3 -: 4], 
                b[4*i+3 -: 4], 
                (i == 0) ? cin : cout_vec[i-1], 
                cout_vec[i], 
                sum[4*i+3 -: 4]
            );
        end
    endgenerate
    assign cout = cout_vec[3];
  endmodule
  ```
### 卡诺图(Karnaugh map to circuit)
- 3变量卡诺图(3-variable Kmap1)
  ```Verilog
  //卡诺图化简不解释
  module top_module(
    input a,
    input b,
    input c,
    output out  ); 
    
    assign out = a | b | c;

  endmodule
  ```
- 4变量卡诺图(4-variable Kmap2)
  ```Verilog
  ///卡诺图圈圈时尽可能大且别遗漏。
    module top_module(
    input a,
    input b,
    input c,
    input d,
    output out  ); 
    
    assign out = ~b & ~c |
        		 b & c & d |
        		 ~a & c & ~d |
        		 a & c & d |
        		 ~a & b & ~d;
  endmodule
  ```
- 4变量卡诺图(4-variable Kmap3)
  ```Verilog
  //这题卡诺图中的d是无关项(也就是01都可以)，为了方便化简就都当1处理了
  module top_module(
    input a,
    input b,
    input c,
    input d,
    output out  ); 
    
    assign out = a | (~b & c);

  endmodule
  ```
- 4变量卡诺图(4-variable Kmap4)
  ```Verilog
  //这题的提示信息可得，输入每变化一次输出就会翻转，典型的异或逻辑
    module top_module(
    input a,
    input b,
    input c,
    input d,
    output out  ); 
    
    assign out = a ^ b ^ c ^ d;

  endmodule
  ```
- 最小项SOP和POS(Minimum SOP and POS)
  ```verilog
    //先求出SOP最小项，然后德摩根定律转成POS形式，最后结果记得求反
    module top_module (
    input a,
    input b,
    input c,
    input d,
    output out_sop,
    output out_pos
  ); 
    assign out_sop = (c & d) | (~a & ~b & c);
    assign out_pos = ~((~c | ~d) & (a | b | ~c));

  endmodule
  ```
- 卡诺图化简(Exam m2014q3)
  ```verilog
    //无聊的化简
  module top_module (
      input [4:1] x, 
      output f );
      
      assign f = (~x[1] & x[3]) | (x[1] & x[2] & ~x[3] & x[4]);

  endmodule
  ```
- 卡诺图化简(Exam 2012q1g)
  ```verilog
    //无聊的化简
  module top_module (
      input [4:1] x,
      output f
  ); 
      assign f = (~x[1] & x[3]) | 
              (~x[2] & ~x[3] & ~x[4]) | 
              (x[1] & ~x[2] & ~x[4]) | 
              (x[2] & x[3] & x[4]);

  endmodule
  ```
- 4选1选择器化简卡诺图(Exam ece241 2014q3)
  ```verilog
    //加了约束的卡诺图化简，只需对每列进行逻辑化简即可
  module top_module (
      input c,
      input d,
      output [3:0] mux_in
  ); 
      assign mux_in[0] = (c | d);
      assign mux_in[1] = 0;
      assign mux_in[2] = ~d;
      assign mux_in[3] = c & d;

  endmodule
  ```
## 时序逻辑(Sequential Logic)
### 锁存器与触发器(Latched and flip-flop)
- D触发器(D flip-flop)
  ```verilog
  module top_module (
    input clk,    // Clocks are used in sequential circuits
    input d,
    output reg q );//

    // Use a clocked always block
    //   copy d to q at every positive edge of clk
    //   Clocked always blocks should use non-blocking assignments
    always @(posedge clk) begin
        q <= d;
    end

  endmodule
  ```
- D触发器(D flip-flops)
  ```verilog
  module top_module (
      input clk,
      input [7:0] d,
      output [7:0] q
  );
      always @(posedge clk)
          q <= d;

  endmodule
  ```
- 电平复位的D触发器(Dff with reset)
  ```verilog
  module top_module (
      input clk,
      input reset,            
      input [7:0] d,
      output [7:0] q
  );
      always @(posedge clk) begin
          if(reset)
              q <= 8'b0;
          else
            q <= d;
      end
  endmodule
  ```
- 电平复位指定值的D触发器(Dff with reset value)
  ```verilog
  module top_module (
      input clk,
      input reset,
      input [7:0] d,
      output [7:0] q
  );
      always @(negedge clk) begin
          if(reset)
              q <= 8'h34;
          else
              q <= d;
      end
  endmodule
  ```
- 异步电平复位的D触发器(Dff with asynchronous reset)
  ```verilog
  //异步的含义就是，将复位电平也加入触发列表中，不用等触发边沿到达后再复位
  module top_module (
      input clk,
      input areset,   // active high asynchronous reset
      input [7:0] d,
      output [7:0] q
  );
      always @(posedge clk or posedge areset) begin
          if(areset)
              q <= 0;
          else
            q <= d;
      end

  endmodule
  ```
- 带写入控制的D触发器(Dff with byte enable)
  ```verilog
  //byteena位控制写入，用case判断下即可
  module top_module (
      input clk,
      input resetn,
      input [1:0] byteena,
      input [15:0] d,
      output [15:0] q
  );
      always @(posedge clk) begin
          if(!resetn)
              q <= 0;
          else begin
              case (byteena)
                  0: ;
                  1: q[7:0] <= d[7:0];
                  2: q[15:8] <= d[15:8];
                  3: q <= d;
              endcase
          end
      end
  endmodule
  ```
- 电平触发的D锁存器(Dff Latch)
  ```verilog
  //电平触发，意味着触发列表任意
  module top_module (
      input d, 
      input ena,
      output q);
      
      always @(*)
          if(ena)
              q <= d;

  endmodule
  ```
- D锁存器(DFF Exams/m2014 q4b)
  ```verilog
  module top_module (
      input clk,
      input d, 
      input ar,   // asynchronous reset
      output q);
      
      always @(posedge clk or posedge ar) begin
          if(ar)
              q <= 0;
          else
              q <= d;
      end

  endmodule
  ```
- D锁存器(DFF Exams/m2014 q4c)
  ```verilog
  module top_module (
      input clk,
      input d, 
      input r,   // synchronous reset
      output q);
      
      always @(posedge clk) begin
          if(r)
              q <= 0;
          else
              q <= d;
      end

  endmodule
  ```
- D锁存器+逻辑门(DFF+Gate Exams/m2014 q4d)
  ```verilog
  module top_module (
      input clk,
      input in, 
      output out);
      
      wire d;
      always @(posedge clk) begin
          d = in ^ out;
          out <= d;
      end
  endmodule
  ```
- 选择器的D触发器(Mux and DFF)
  ```Verilog
  //注意这题是子模块的硬件描述，另外值得注意的是如果用中间变量wire d <=L ? r_in : q_in; Q <= d;会产生错误的结果
  module top_module (
	input clk,
	input L,
	input r_in,
	input q_in,
	output reg Q);

    always @(posedge clk )
        Q= L ? r_in : q_in;
    
  endmodule
  ```
- 两级选择器的D触发器(mt2015 MuxDFF)
  ```Verilog
  //仍然注意不要再always模块内产生wire中间变量
  module top_module (
      input clk,
      input w, R, E, L,
      output Q
  );
      always @(posedge clk ) begin
          Q <= L ? R : (E ? w : Q);
      end

  endmodule
  ```
- 门电路与D触发器(DFF and gates Exams/ece241 2014 q4)
  ```Verilog
  // 注意第二位第三位状态反馈的是~Q
  module top_module (
      input clk,
      input x,
      output z
  ); 
      reg [2:0] state;
      always @(posedge clk ) begin
          state[0] <= x ^ state[0];
          state[1] <= x & ~state[1];
          state[2] <= x | ~state[2];
      end
      assign z = ~(|state);

  endmodule
  ```
- JK触发器(Create circuit from truth table)
  ```Verilog
  // 根据需求真值表画出卡诺图得出Q*和Q的迭代逻辑
  module top_module (
    input clk,
    input j,
    input k,
    output Q); 

    always @(posedge clk ) begin
        Q <= (~k & Q) | (j & ~Q);
    end

  endmodule
  ```
- 边沿检测(Edge detect)
  ```Verilog
  // 这题明显需要用个记忆变量reg来存储此前的状态
  module top_module (
      input clk,
      input [7:0] in,
      output reg [7:0] pedge
  );

      reg [7:0] in_prev;  // 存储上一个时钟周期的输入值
      always @(posedge clk) begin
          // 检测上升沿
          pedge <= in & ~in_prev;
          // 更新 in_prev
          in_prev <= in;
      end

  endmodule
  ```
- 边沿检测2(Edge detect2)
  ```Verilog
  // 输入只要发生翻转输出就为1，典型的异或逻辑
  module top_module (
      input clk,
      input [7:0] in,
      output [7:0] anyedge
  );
      
      reg [7:0] prev_in;

      always @(posedge clk) begin
          anyedge <= in ^ prev_in;
          prev_in <= in;
      end

  endmodule
  ```
- 边缘捕获(Edge capture)
  ```Verilog
  // 这题我卡了好一会儿，参考了网上其他朋友的答案才想通的
  module top_module (
      input clk,
      input reset,
      input [31:0] in,
      output [31:0] out
  );
      reg [31:0] prev_in;
      always @(posedge clk ) begin
          if(reset) begin
              out <= 32'b0;
          end
          else
              //关键逻辑：检测到下降沿时置1，未检测到下降沿时候保持
              out <= (prev_in & ~in) | out;  
          prev_in <= in;  //prev_in更新必须写在always层级块
      end

  endmodule
  ```
- 双边沿D触发器(Dual-edge trigger flip-flop)
  ```Verilog
  // 这题没做出来，直接参考的他人答案思路，官网的题解思路跟这不一样
  module top_module (
      input clk,
      input d,
      output q
  );
      reg q1,q2;
      // 上升下降沿都捕获并存储到临时变量q1,q2
      always @(posedge clk)
          q1 <= d;
      always @(negedge clk)
          q2 <= d;
      // 时钟回到高电平就是q1低电平为q2
      assign q = clk ? q1 : q2;
  endmodule
  ```

### 计数器(Counters)
- 4位计数器(4-bit binary counter)
  ```Verilog
  //
  module top_module (
      input clk,
      input reset,      // Synchronous active-high reset
      output [3:0] q);

      always @(posedge clk ) begin
          if(reset)
              q <= 4'b0000;
          else
              q <= q + 1;
      end
  endmodule
  ```
- 10进制计数器(Count10)
  ```Verilog
  //注意是逢9复位而不是逢10再复位
  module top_module (
      input clk,
      input reset,        // Synchronous active-high reset
      output [3:0] q);

      always @(posedge clk ) begin
          if(reset || q == 4'b1001)
              q <= 4'b0000;
          else
              q <= q + 1;
      end

  endmodule
  ```
- 1-10计数器(Count1to10)
  ```Verilog
  // 注意虽然是从1开始复位了，但仍然是十进制计数器，所以得逢10复位了
  module top_module (
      input clk,
      input reset,
      output [3:0] q);

      always @(posedge clk ) begin
          if(reset || q == 4'b1010)
              q <= 4'b0001;
          else
              q <= q + 4'b0001;
      end
  endmodule
  ```
- 慢计数器(CountSlow)
  ```Verilog
  // 注意这题的slowena输入不但控制着q自增，从9回到0也算计数器自增逻辑
  module top_module (
      input clk,
      input slowena,
      input reset,
      output [3:0] q);

      always @(posedge clk ) begin
          if(reset)
              q <= 4'b0000;
          else
              if(slowena)
                  if(q == 4'b1001)
                      q <= 4'b0000;
                  else
                      q <= q + 1;
      end
  endmodule
  ```
- 1-12计数器(Count1-12)
  ```Verilog
  // 始终无法理解这题的答案思路，仅贴上参考他人的正确答案
  module top_module (
      input clk,
      input reset,
      input enable,
      output [3:0] Q,
      output c_enable,
      output c_load,
      output [3:0] c_d
  ); //
      
      assign c_enable = enable;
      assign c_load = reset | ((Q == 4'd12) && enable == 1'b1);
      assign c_d = c_load ? 4'd1 : 4'd0;
  
      count4 u_counter (clk, c_enable, c_load, c_d, Q);
  
  endmodule
  ```
- 1000BCD计数器(Counter 1000)
  ```Verilog
  //第一反应是计数器级联实现，但题目格式限定了只能调用已有的BCD计数器来实现
  module top_module (
      input clk,
      input reset,
      output OneHertz,
      output [2:0] c_enable
  ); //

      reg [3:0] one, ten, hundred;
      assign c_enable[0] = 1;
      assign c_enable[1] = one == 9;
      assign c_enable[2] = (ten == 9 && one == 9);
      assign OneHertz = (hundred == 9 && ten == 9 && one == 9);

      bcdcount bcd1(clk, reset, c_enable[0], one);
      bcdcount bcd10(clk, reset, c_enable[1], ten);
      bcdcount bcd100(clk, reset, c_enable[2], hundred);

  endmodule
  ```
- 4位BCD码计数器(4-digit decimal counter)
  ```Verilog
  // 这题是目前为止花了最长时间的，思路虽简单但细节坑太多了，对Verilog语法理解稍有偏颇以及一些细节处理不到位就会通不过，最后是靠着AI慢慢写对的
  module bcd_adder4(
      input [3:0] a,b,
      input cin,
      output [3:0] sum,
      output cout
    );
      wire[4:0] temp_sum;
      assign temp_sum = a + b + cin;
      assign cout = temp_sum > 4'd9 ? 1 : 0;
      assign sum = temp_sum > 4'd9 ? 
            temp_sum + 4'd6 : temp_sum;
  endmodule


  module top_module (
      input clk,
      input reset,   // Synchronous active-high reset
      output [3:1] ena,
      output reg [15:0] q
  );

      wire [3:0] sum_ones, sum_tens, sum_hundreds, sum_thousand;
      wire ena_1, ena_2, ena_3;

      // 实例化 BCD 加法器，计算下一时钟周期的值
      bcd_adder4 one_inc(q[3:0], 4'b1, 1'b0, sum_ones, ena_1);
      bcd_adder4 ten_inc(q[7:4], 4'b0, ena_1, sum_tens, ena_2);
      bcd_adder4 hundred_inc(q[11:8], 4'b0, ena_2, sum_hundreds, ena_3);
      bcd_adder4 thousand_inc(q[15:12], 4'b0, ena_3, sum_thousand, );

      assign ena = {ena_3, ena_2, ena_1}; // 进位信号

      always @(posedge clk) begin
          if (reset)
              q <= 16'b0;
          else begin
              q[3:0] <= sum_ones;      // 存储计算出的BCD个位
              q[7:4] <= sum_tens;      // 存储计算出的BCD十位
              q[11:8] <= sum_hundreds; // 存储计算出的BCD百位
              q[15:12] <= sum_thousand; // 存储计算出的BCD千位
          end
      end

  endmodule
  ```
- 12小时时钟(12-hour clock)
  ```Verilog
  //
  module top_module(
      input clk,
      input reset,
      input ena,
      output reg pm,
      output reg [7:0] hh,
      output reg [7:0] mm,
      output reg [7:0] ss); 

      wire [7:0] hour, min, sec;
      wire sec_carry, min_carry, hour_carry;
      bcd60counter sec_counter(sec, 1, sec, sec_carry);
      bcd60counter min_counter(min, sec_carry, min, min_carry);
      bcd12counter hour_counter(hour, min_carry, hour, hour_carry);

      always @(posedge clk ) begin : clock_tick
          if(reset) begin
              hh <= 8'h12;
              mm <= 8'd0;
              ss <= 8'd0;
              pm <= 0;
          end
          else if(ena) begin
              ss <= sec;
              mm <= min;
              hh <= hour;
              
              if(hour_carry) begin
                  hh <= pm ? 1 : 0;
                  pm <= ~pm;
              end
              else
                  hh <= hour;
          end
      end
  endmodule

  module one_bcd_inc(
      input [3:0] a, b,
      output [3:0] inc_out,
      output cout
  );
      wire[4:0] temp_sum;
      assign temp_sum = a + b;
      assign cout = temp_sum > 5'd9 ? 1 : 0;
      assign inc_out = temp_sum > 5'd9 ? 
          temp_sum + 5'd6 : temp_sum;
  endmodule


  module bcd12counter(
      input [7:0] inc_in,
      input cin,
      output [7:0] inc_out,
      output cout
  );
      wire[3:0] bcd_ten;
      one_bcd_inc inc(inc_in[3:0], cin, bcd_ten,);
      assign inc_out = {bcd_ten, inc_in[3:0]};
      assign cout = inc_out == 8'h12;
  endmodule

  module bcd60counter(
      input [7:0] inc_in,
      input cin,
      output [7:0] inc_out,
      output cout
  );
      wire[3:0] bcd_ten;
      one_bcd_inc inc(inc_in[3:0], cin, bcd_ten,);
      assign inc_out = {bcd_ten, inc_in[3:0]};
      assign cout = inc_out == 8'h60;
  endmodule
  ```
### 移位寄存器(Shift Registers)
- 4位右移寄存器(shift4)
  ```Verilog
  // 这题注意Verilog语法中always块内不允许对input变量进行赋值
  module top_module(
      input clk,
      input areset,  // async active-high reset to zero
      input load,
      input ena,
      input reg [3:0] data,
      output reg [3:0] q); 

      always @(posedge clk or posedge areset) begin
          if(areset)
              q <= 4'b0000;
          else begin
              if(load == 1)
                  q <= data;
              else if(ena == 1)
                  q <= q >> 1;
          end
      end
  endmodule

  ```
- 左右循环移动寄存器(left/right rotator)
  ```Verilog
  // 本以为会有循环移位的Verilog语法，得知没有后只能用分号重组向量
  module top_module(
      input clk,
      input load,
      input [1:0] ena,
      input [99:0] data,
      output reg [99:0] q); 
      
      always @(posedge clk ) begin
          if (load) q <= data;
          else begin
              case (ena)
                  1 : q <= {q[0], q[99:1]};
                  2 : q <= {q[98:0], q[99]};
                  default: q <= q;
              endcase
          end
      end
  endmodule

  ```
- 左右算术移位寄存器
  ```Verilog
  // 
  module top_module(
      input clk,
      input load,
      input ena,
      input [1:0] amount,
      input [63:0] data,
      output reg [63:0] q); 

      always @(posedge clk ) begin
          if(load)
              q <= data;
          else
              if(ena) begin
                  case(amount)
                      2'b00: q <= q << 1;
                      2'b01: q <= q << 8;
                      2'b10: q <= {q[63], q[63:1]};
                      2'b11: q <= {{8{q[63]}}, q[63:8]};
                  endcase
              end
      end
  endmodule
  ```
- 5位线性反馈移位寄存器(5-bit LFSR)
  ```Verilog
  // 开始进入有限状态机内容了
  module top_module(
      input clk,
      input reset,    // Active-high synchronous reset to 5'h1
      output [4:0] q
  ); 

      always @(posedge clk ) begin
          if(reset)
              q <= 5'h1;
          else begin
              q[4] <= q[0];
              q[3] <= q[4];
              q[2] <= q[3] ^ q[0];
              q[1] <= q[2];
              q[0] <= q[1];
          end
      end
  endmodule
  ```
- 3位LFSR(3-bit LFSR)
  ```Verilog
  // 注意这题的clk信号是KEY[1]
  module top_module (
    input [2:0] SW,      // R
    input [1:0] KEY,     // L and clk
    output [2:0] LEDR);  // Q

      always @(posedge KEY[0]) begin
          if(KEY[1] == 1)
              LEDR <= SW;
          else begin
              LEDR[2] <= LEDR[2] ^ LEDR[1];
              LEDR[1] <= LEDR[0];
              LEDR[0] <= LEDR[2];
          end
      end
  endmodule

  ```
- 32位LFSR(32-bit LFSR)
  ```Verilog
  // 这题用到了5位LFSR题解的技巧，先全体正常移位，对Tab特殊位进行逻辑描述后送赋值给一个临时变量q_next，后续就用q_next更新
  module top_module(
      input clk,
      input reset,    // Active-high synchronous reset to 32'h1
      output [31:0] q //32,22,2,1 tab
  ); 

      reg [31:0] q_next;

      always @(*) begin
          q_next <= q[31:1];
          q_next[31] <= q[0];
          q_next[21] <= q[22] ^ q[0];
          q_next[1] <= q[2] ^ q[0];
          q_next[0] <= q[1] ^ q[0];
      end

      always @(posedge clk ) begin
          if(reset)
              q <= 32'h1;
          else begin
              q <= q_next;
          end
      end
  endmodule
  ```
- 移位寄存器(Shift register Exam/m2014 q4k)
  ```Verilog
  // 这题看着简单但是细节和坑很多
  module top_module (
      input clk,
      input resetn,   // synchronous reset
      input in,
      output out);

      reg[3:0] reg4;
      always @(posedge clk ) begin
          if(resetn == 0)
              reg4 <= 0;
          else
              reg4 <= {in, reg4[3:1]};
      end
      assign out = reg4[0];

  endmodule

  ```
- 移位寄存器(Shift register)
  ```Verilog
  // 这题复用此前Mux寄存器的模块，把模块参数传对线连好就行
    module top_module (
    input [3:0] SW,
    input [3:0] KEY,
    output [3:0] LEDR
    ); //

        MUXDFF MUXDFF_0 (
            .clk(KEY[0]),
            .E(KEY[1]),
            .L(KEY[2]),
            .w({KEY[3],LEDR[3:1]}),
            .R(SW),
            .q(LEDR)
        );

    endmodule

    module MUXDFF (
        input clk,
        input E,
        input L,
        input [3:0] w,
        input [3:0] R,
        output reg [3:0] q
    );

        always @(posedge clk ) begin
            q <= L ? R : (E ? w : q);
        end

    endmodule
  ```
- 3输入LUT(3-input LUT)
  ```Verilog
  // Z = Q[{A,B,C}]开始我是用case报错通不过，后来发现有这一Verilog语法糖
    module top_module (
        input clk,
        input enable,
        input S,
        input A, B, C,
        output Z ); 

        reg [7:0] Q;

        always @(posedge clk ) begin
            if(enable)
                Q <= {Q[6:0], S};
        end
        assign Z = Q[{A,B,C}];
    endmodule
  ```
### 更多电路(More circuits)
- 规则90(Rule 90)
  ```Verilog
  // 根据题目描述实现一个错位异或功能
    module top_module(
        input clk,
        input load,
        input [511:0] data,
        output [511:0] q ); 

        always @(posedge clk ) begin
            if(load)
                q <= data;
            else
                q <= {q[510:0], 1'b0} ^ {1'b0, q[511:1]};
        end
    endmodule
  ```
- 规则110(Rule 110)
  ```Verilog
  // 这题思路秒得，但排错查到吐血，因为把移位方向搞反了！
    module top_module(
        input clk,
        input load,
        input [511:0] data,
        output [511:0] q ); 

        wire[511:0] left, right;
        assign left = {q[510:0], 1'b0};
        assign right = {1'b0, q[511:1]};

        always @(posedge clk ) begin
            if(load)
                q <= data;
            else
                q <= (~right & (q | left)) | (q ^ left);
        end
    endmodule
  ```
- Conway's Game of life 16x16
  ```Verilog
  // Verilog写出了Leetcode题的感觉，题目思路很简单，对这种环绕网格，处理关键就是对超出范围的索引进行取模实现环绕效果(类似于Python的索引),主要是通过题目熟悉fucntion调用等Verilog语法细节问题
    module top_module(
        input clk,
        input load,
        input [255:0] data,
        output [255:0] q ); 

        integer neighbours;

        always @(posedge clk ) begin
            if(load)
                q <= data;
            else begin
                for (int row=0; row<16; row++) begin
                    for(int col=0; col<16; col++) begin
                        neighbours = neighbour_cnt(q, row, col);
                        if(neighbours < 2 || neighbours > 3)
                            q[16*row+col] <= 0;
                        else if(neighbours == 3)
                            q[16*row+col] <= 1;
                        else
                            continue;
                    end
                end
            end
        end

    endmodule

    function [2:0] neighbour_cnt;
        input [255:0] grid;
        input [3:0] row, col;
        
        integer l,r,u,d;
        begin
            neighbour_cnt = 0;
            l = wrap_idx(col-1);
            r = wrap_idx(col+1);
            u = wrap_idx(row-1);
            d = wrap_idx(row+1);
            row = wrap_idx(row);
            col = wrap_idx(col);
            neighbour_cnt = 
                grid[16*u + l] + grid[16*u + col] + grid[16*u + r] +
                grid[16*row + l]                  +grid[16*row + r] + 
                grid[16*d + l] + grid[16*d + col] + grid[16*d + r];
        end    

    endfunction

    function [3:0] wrap_idx;
        input [3:0] idx;

        begin
            wrap_idx = (idx + 16) % 16;
        end
        
    endfunction
  ```
### 有限状态机(Finite state machine)
- 简单异步有限状态机(FSM1 async reset)
  ```Verilog
  // 这题只有个单一状态的翻转，思路很简单in为0翻转，in为1保持，然而需要注意next_state需要赋初值,否则综合会生成Latch导致不符预期的电路行为
    module top_module(
        input clk,
        input areset,    // Asynchronous reset to state B
        input in,
        output out);//  

        parameter A=0, B=1; 
        reg state, next_state;

        always @(*) begin    // This is a combinational always block
            // State transition logic
            next_state <= state;
            if(in == 0)
                next_state <= ~state;
        end

        always @(posedge clk, posedge areset) begin    // This is a sequential always block
            // State flip-flops with asynchronous reset
            if(areset)
                state <= B;
            else
                state <= next_state;
        end
        // Output logic
        // assign out = (state == ...);
        assign out = state;
    endmodule
  ```
- 简单同步状态机(FSM1 sync reset)
  ```Verilog
  // 这题务必弄清楚阻塞赋值与非阻塞赋值的根本区别，此前都是感觉always块内对reg变量就无脑用<=赋值,always块外的wire变量无脑=赋值，基础不牢地动山摇在这题体现非常明显
    // Note the Verilog-1995 module declaration syntax here:
    module top_module(clk, reset, in, out);
    input clk;
    input reset;    // Synchronous reset to state B
    input in;
    output out;//  
    reg out;

    // Fill in state name declarations

    parameter A = 0, B = 1;
    reg present_state, next_state;

    always @(posedge clk) begin
        if (reset) begin  
            // Fill in reset logic
            present_state <= B;// Fill in reset state
            out <= B;
        end else begin
            case (present_state)
                A: next_state = in ? A : B;
                B: next_state = in ? B : A;
                default: next_state = present_state;
                // Fill in state transition logic
            endcase

            // State flip-flops
            present_state = next_state;   

            case (present_state)
                // Fill in output logic
                default:out = present_state;
            endcase
        end
    end

    endmodule
  ```
- 简单异步状态机2(Simple async FSM2)
  ```Verilog
  // 根据状态图写出真值表化简得出这是JK触发器逻辑，三段式一步到位
    module top_module(
        input clk,
        input areset,    // Asynchronous reset to OFF
        input j,
        input k,
        output out); //  

        parameter OFF=0, ON=1; 
        reg state, next_state;

        always @(*) begin
            // State transition logic
            next_state = (j & ~state) | (~k & state);
        end

        always @(posedge clk, posedge areset) begin
            // State flip-flops with asynchronous reset
            if(areset)
                state <= OFF;
            else
                state <= next_state;
        end

        assign out = state;

        // Output logic
        // assign out = (state == ...);

    endmodule
  ```
- 简单同步状态机2(Simple sync FSM2)
  ```Verilog
  //除了触发方式变了外，就是上一题的复制粘贴
    module top_module(
        input clk,
        input reset,    // Asynchronous reset to OFF
        input j,
        input k,
        output out); //  

        parameter OFF=0, ON=1; 
        reg state, next_state;

        always @(*) begin
            // State transition logic
            next_state = (j & ~state) | (~k & state);
        end

        always @(posedge clk) begin
            // State flip-flops with asynchronous reset
            if(reset)
                state <= OFF;
            else
                state <= next_state;
        end

        assign out = state;

        // Output logic
        // assign out = (state == ...);

    endmodule
  ```
- 简单状态转移3(Simple State transitions3)
  ```Verilog
  // 这题仅仅需要构建状态间的映射，并不需要时钟信号和触发器来迭代状态
    module top_module(
        input in,
        input [1:0] state,
        output [1:0] next_state,
        output out); //

        parameter A=0, B=1, C=2, D=3;

        // State transition logic: next_state = f(state, in)
        always @(*) begin
            case(state)
                A: next_state = in ? B : A;
                B: next_state = in ? B : C;
                C: next_state = in ? D : A;
                D: next_state = in ? B : C;
            endcase
        end
        // Output logic:  out = f(state) for a Moore state machine
        assign out = (state == D);

    endmodule
  ```
- 独热编码的状态转移3(one-hot state transition 3)
  ```Verilog
  // 这题提供了一个状态转移的独热编码方式，常规FSM设计都是根据N个状态来确定触发器个数Log2 N,然而独热编码是N个状态直接用N位表示，一个状态独占特定位为1，其他位为0，这样做的好处是省去了状态机设计时的复杂组合逻辑，代价就是实现状态机电路的触发器个数变为了N
    module top_module(
        input in,
        input [3:0] state,
        output [3:0] next_state,
        output out); //

        parameter A=0, B=1, C=2, D=3;

        // State transition logic: Derive an equation for each state flip-flop.
        assign next_state[A] = ~in &(state[A] | state[C]);
        assign next_state[B] = in &(state[A] | state[B] | state[D]);
        assign next_state[C] = ~in & (state[B] | state[D]);
        assign next_state[D] = in & state[C];

        // Output logic: 
        assign out = state[D];

    endmodule
  ```
- 异步复位的简单状态转移机3(FSM 3)
  ```Verilog
  // 增加了迭代步骤，使用两个D触发器实现该状态机
    module top_module(
        input clk,
        input in,
        input areset,
        output out); //

        parameter A=2'b00, B=2'b01, C=2'b10, D=2'b11;
        reg [1:0] state, next_state;

        // State transition logic
        always @(*) begin
            case(state)
                A: next_state = in ? B : A;
                B: next_state = in ? B : C;
                C: next_state = in ? D : A;
                D: next_state = in ? B : C;
            endcase
        end

        // State flip-flops with asynchronous reset
        always @(posedge clk , posedge areset) begin
            if(areset)
                state <= A;
            else
                state <= next_state;
        end

        // Output logic
        assign out = state == D;

    endmodule
  ```
- 同步复位的简单状态转移机3(FSM3 sync)
  ```Verilog
  // 搞不懂作者好多同一道题都要弄个同步异步，除了改动触发列表啥区别都没有。
    module top_module(
        input clk,
        input in,
        input reset,
        output out); //

        parameter A=2'b00, B=2'b01, C=2'b10, D=2'b11;
        reg [1:0] state, next_state;

        // State transition logic
        always @(*) begin
            case(state)
                A: next_state = in ? B : A;
                B: next_state = in ? B : C;
                C: next_state = in ? D : A;
                D: next_state = in ? B : C;
            endcase
        end

        // State flip-flops with asynchronous reset
        always @(posedge clk) begin
            if(reset)
                state <= A;
            else
                state <= next_state;
        end

        // Output logic
        assign out = state == D;

    endmodule
  ```
- 设计一个Moore有限状态机(Design a Moore FSM)
  ```Verilog
  // 这题我也是参考网友答案的，但他那个版本代码太多冗余逻辑了进行了化简，起初我对这题状态建模理解有误，正确理解应该是水位状态0代表最低，3代表最高总计有4种状态，下一次水位状态取决于已开启的传感器输入，阀门输出fr1~fr3取决于当前水位状态，dfr输出取决于前后两次水位升降还是持平，所以需要一个寄存器dfr_reg保持更新
    module top_module (
        input clk,
        input reset,
        input [3:1] s,
        output fr3,
        output fr2,
        output fr1,
        output dfr
    ); 
        reg [1:0] level, next_level;
        reg dfr_reg;

        // 计算 next_level
        always @(*) begin
            next_level = vec_ones(s);
        end

        // 状态更新
        always @(posedge clk ) begin
            if(reset)
                level <= 0;
            else
                level <= next_level;
        end

        // fr1, fr2, fr3 输出
        assign {fr1, fr2, fr3} = (3'b111 << level);

        // dfr 输出
        assign dfr = dfr_reg;

        always @(posedge clk) begin
            if (reset) 
                dfr_reg <= 1'b1; // 复位到 s0
            else if (next_level > level)
                dfr_reg <= 1'b0; // 只有 level 上升时 dfr 才变为 0
            else if (next_level < level)
                dfr_reg <= 1'b1; // 下降时恢复到 1
        end

    endmodule

    // 计算 s[3:1] 中 1 的个数
    function automatic [1:0] vec_ones;
        input [3:1] vec;
        integer i;
        begin
            vec_ones = 0;
            for (i = 1; i <= 3; i = i + 1) // 遍历 s[3:1]
                if (vec[i])
                    vec_ones = vec_ones + 1;
        end
    endfunction
  ```
- 游戏Lemmings 1(Lemmings 1)
  ```Verilog
  // 理解题意后画出状态转移图+真值表，然后FSM三段式得解
    module top_module(
        input clk,
        input areset,    // Freshly brainwashed Lemmings walk left.
        input bump_left,
        input bump_right,
        output walk_left,
        output walk_right); //  

        parameter LEFT=0, RIGHT=1;
        reg state, next_state;

        always @(*) begin
            // State transition logic
            case(state)
                LEFT: next_state = bump_left ? RIGHT : LEFT;
                RIGHT: next_state = bump_right ? LEFT : RIGHT;
                default: next_state = LEFT;
            endcase
        end

        always @(posedge clk, posedge areset) begin
            // State flip-flops with asynchronous reset
            if(areset)
                state <= LEFT;
            else
                state <= next_state;
        end

        // Output logic
        assign walk_left = (state == LEFT);
        assign walk_right = (state == RIGHT);

    endmodule
  ```
- 游戏Lemmings 2(Lemmings 2)
  ```Verilog
  // 这题多了一个跌落状态，多画个状态图节点和转移条件而已，注意用一个寄存器dir_mem记录并恢复跌落前时候的行进方向。
    module top_module(
        input clk,
        input areset,    // Freshly brainwashed Lemmings walk left.
        input bump_left,
        input bump_right,
        input ground,
        output walk_left,
        output walk_right,
        output aaah ); 

        reg [1:0] state, next_state, dir_mem;
        parameter left = 0, right = 1, fall = 2;

        always @(*) begin
            if(ground) begin
                case(state)
                    left: begin
                        next_state = bump_left ? right : left;
                        dir_mem = state;
                    end 
                    right: begin
                        next_state = bump_right ? left : right;
                        dir_mem = state;
                    end
                    fall: next_state = dir_mem;

                    default: begin
                        next_state = left;
                        dir_mem = state;
                    end
                endcase
            end
            else
                next_state = fall;
        end

        always @(posedge clk or posedge areset)
            if (areset)
                state <= left;
            else begin
                state <= next_state;
            end
                
        assign walk_left = (state == left);
        assign walk_right = (state == right);
        assign aaah = (state == fall);

    endmodule
  ```
- 游戏Lemmings 3(Lemmings 3)
  ```Verilog
  // 这题又多了一个挖掘状态，状态和约束一多逻辑就容易乱，自己本以为if else块完美无暇的逻辑还是有漏洞，最后在AI的帮助下改对，还是老老实实最笨的case穷举最靠谱
    module top_module(
        input clk,
        input areset,    // Freshly brainwashed Lemmings walk left.
        input bump_left,
        input bump_right,
        input ground,
        input dig,
        output walk_left,
        output walk_right,
        output aaah,
        output digging ); 

        reg [1:0] state, next_state, dir_mem;
        parameter left = 0, right = 1, fall = 2, digg = 3;

        always @(*) begin
            case (state)
                left: begin
                    if (!ground)
                        next_state = fall;  // 若地面丢失，转入掉落
                    else if (dig)
                        next_state = digg;  // 挖掘优先级高于转向
                    else
                        next_state = bump_left ? right : left;
                end
                
                right: begin
                    if (!ground)
                        next_state = fall;
                    else if (dig)
                        next_state = digg;
                    else
                        next_state = bump_right ? left : right;
                end

                fall:
                    next_state = ground ? dir_mem : fall; // 掉落结束后恢复行走状态

                digg:
                    next_state = ground ? digg : fall; // 掉落结束后恢复行走状态

                default: next_state = left;
            endcase
        end

        always @(posedge clk or posedge areset) begin
            if (areset)
                state <= left;
            else begin
                if (state == left || state == right)
                    dir_mem <= state;  // 只在行走时存储方向
                state <= next_state;
            end
        end

        assign walk_left = (state == left);
        assign walk_right = (state == right);
        assign aaah = (state == fall);
        assign digging = (state == digg);

    endmodule
  ```
- 游戏Lemmings 4(Lemmings 4)
  ```Verilog
  // 每多一个状态思路不难，但是细节和坑又多一堆。这题最大的坑就是空中计时逻辑，原来是if(state == fall)开启计时通过不了仿真，但if(!ground)开启计时却又正确了
    module top_module(
        input clk,
        input areset,    // Freshly brainwashed Lemmings walk left.
        input bump_left,
        input bump_right,
        input ground,
        input dig,
        output walk_left,
        output walk_right,
        output aaah,
        output digging ); 

        parameter left = 0, right = 1, fall = 2, digg = 3, splater = 4;
        reg [2:0] state, next_state, dir_mem = left;
        reg [31:0] air_time;
        
        always @(*) begin
            case (state)
                left: begin
                    if (!ground)
                        next_state = fall;  // 若地面丢失，转入掉落
                    else if (dig)
                        next_state = digg;  // 挖掘优先级高于转向
                    else
                        next_state = bump_left ? right : left;
                end
                
                right: begin
                    if (!ground)
                        next_state = fall;
                    else if (dig)
                        next_state = digg;
                    else
                        next_state = bump_right ? left : right;
                end

                fall: begin
                    if(ground)
                        next_state = (air_time > 20) ? splater : dir_mem;
                    else
                        next_state = fall;
                end
                digg:
                    next_state = ground ? digg : fall; // 掉落结束后恢复行走状态
                
                splater:
                    next_state = splater;  // 摔死后不再改变状态

                default: next_state = left;
            endcase
        end

        always @(posedge clk or posedge areset) begin
            if (areset) begin
                state <= left;
                air_time <= 0;
            end
            else begin
                if (state == left || state == right)
                    dir_mem <= state;  // 只在行走时存储方向
                if (!ground)
                    air_time <= air_time + 1;  // 记录掉落时间
                else
                    air_time <= 0;
                state <= next_state;
            end
        end

        assign {walk_left, walk_right, aaah, digging} = 
        {state == left, state == right, state == fall, state == digg};

    endmodule
  ```
- 有限状态机独热码(FSM onehot)
  ```Verilog
  //
    module top_module(
        input in,
        input [9:0] state,
        output [9:0] next_state,
        output out1,
        output out2);

        parameter   S0 = 10'b0000000001,
                    S1 = 10'b0000000010,
                    S2 = 10'b0000000100,
                    S3 = 10'b0000001000,
                    S4 = 10'b0000010000,
                    S5 = 10'b0000100000,
                    S6 = 10'b0001000000,
                    S7 = 10'b0010000000,
                    S8 = 10'b0100000000,
                    S9 = 10'b1000000000;

        always @(*) begin
            case(state)
                S0: next_state = in ? S1 : S0;
                S1: next_state = in ? S2 : S0;
                S2: next_state = in ? S3 : S0;
                S3: next_state = in ? S4 : S0;
                S4: next_state = in ? S5 : S0;
                S5: next_state = in ? S6 : S8;
                S6: next_state = in ? S7 : S9;
                S7: next_state = in ? S7 : S0;
                S8: next_state = in ? S1: S0;
                S9: next_state = in ? S1 : S0;            
                default: next_state = S0;
            endcase
        end

        assign out1 = (state == S9 ||state == S8);
        assign out2 = (state == S9 || state == S7);

    endmodule
  ```
- PS/2解包(PS/2 package parser)
  ```Verilog
  // 如果不是看提示，这题用自己的状态建模怎么都不正确，清晰明了的状态建模非常重要
    module top_module(
        input clk,
        input [7:0] in,
        input reset,    // Synchronous reset
        output done); //

        parameter DONE = 0, BYTE1 = 1, BYTE2 = 2, BYTE3 = 3;
        reg[1:0] state, next_state;

        // State transition logic (combinational)
        always @(*) begin
            case(state)
                DONE: next_state = in[3] ? BYTE2 : BYTE1;
                BYTE1: next_state = in[3] ? BYTE2 : BYTE1;
                BYTE2: next_state = BYTE3;
                BYTE3: next_state = DONE;
                default: next_state = BYTE1;
            endcase
        end

        // State flip-flops (sequential)
        always @(posedge clk ) begin
            if(reset)
                state <= BYTE1;
            else
                state <= next_state;
        end
    
        // Output logic
        assign done = (state == DONE);
        
    endmodule
  ```
- PS/2解包与数据路径(PS/2 package parser and datapath)
  ```Verilog
  // 本以为能一次通过，结果编译试错了好多遍才过的，很多细节例如存储的顺序，触发的先后时机，好在最后看波形排查还是过了。
    module top_module(
        input clk,
        input [7:0] in,
        input reset,    // Synchronous reset
        output [23:0] out_bytes,
        output done);

        parameter BYTE1 = 1, BYTE2 = 2, BYTE3 = 3, DONE = 0;
        reg [1:0] state, next_state;
        reg [23:0] reg_out_bytes;
        
        // FSM from fsm_ps2
        always @(*) begin
            case(state)
                DONE: next_state = in[3] ? BYTE2 : BYTE1;
                BYTE1: next_state = in[3] ? BYTE2 : BYTE1;
                BYTE2: next_state = BYTE3;
                BYTE3: next_state = DONE;
                default: next_state = BYTE1;
            endcase
        end

        always @(posedge clk) begin
            if(reset)
                state <= BYTE1;
            else begin
                case(state)
                    BYTE1: reg_out_bytes[23:16] <= in;
                    BYTE2: reg_out_bytes[15:8] <= in;
                    BYTE3: reg_out_bytes[7:0] <= in;
                    DONE: reg_out_bytes[23:16] <= in;
                endcase
                state <= next_state;
            end
        end

        assign done = state == DONE;
        assign out_bytes = reg_out_bytes;

        // New: Datapath to store incoming bytes.

    endmodule

  ```
- 串口接收器(Serial receiver)
  ```Verilog
  // 这一题思路跟前两题思路类似，只不过多了起始位和终止位来控制状态的转移，让人疑惑的是计数逻辑如果和状态更新逻辑放在一个always块内会通过不了仿真，但分离却能通过仿真，可能next_state更新和计数逻辑存在耦合。
    module top_module(
        input clk,
        input in,
        input reset,    // Synchronous reset
        output done
    ); 
        parameter idle = 0, start = 1, sending = 2, stop = 3, error = 4;
        reg [2:0] state, next_state;
        int rcved_bits;
        reg done_r;

        always @(*) begin
            case(state)
                idle: next_state = in ? idle : start;
                
                start: next_state = sending;

                sending:next_state = (rcved_bits == 8) ?
                    in ? stop : error : sending;

                error: next_state = in ? idle : error;
                
                stop:
                    next_state = in ? idle : start;

                default:
                    next_state = idle;
            endcase
        end

        always @(posedge clk) begin
            if(reset)
                state <= idle;
            else
                state <= next_state;
        end

        always @(posedge clk ) begin
            if(reset)
                rcved_bits <= 0;
            else
                case(next_state)
                    start: rcved_bits <= 0;
                    sending:
                        rcved_bits <= rcved_bits + 1;
                    default:;
                endcase
        end

        assign done = state == stop;

    endmodule

  ```
- 串口接收器与数据路径(Serial receiver and datapath)
  ```Verilog
  // 在上题基础上加一个data寄存器用于记录sending过程中的数据，哪怕思路简单也很难一次通过，慢慢看波形调试几遍才过的，注意data的更新顺序。
    module top_module(
        input clk,
        input in,
        input reset,    // Synchronous reset
        output [7:0] out_byte,
        output done
    ); 
    parameter idle = 0, start = 1, sending = 2, stop = 3, error = 4;
    reg [2:0] state;
    reg [2:0] next_state;
    int bit_count;
    reg [7:0] data;

    always @(*) begin
        case(state)
            idle: 
                next_state = in ? idle : start;

            start: next_state = sending;

            sending: begin
                if(bit_count == 8)
                    next_state = in ? stop : error;
                else
                    next_state = sending;
            end

            stop:
                next_state = in ? idle : start;
            
            error:
                next_state = in ? idle : error;
            default: next_state = idle;
        endcase
    end

    always @(posedge clk ) begin
        if(reset)
            state <= idle;
        else
            state <= next_state;
    end

    always @(posedge clk) begin
        if(reset)
            bit_count <= 0;
        else
            case(next_state)
                start: bit_count <= 0;
                sending: begin
                    bit_count <= bit_count + 1;
                    data <= {in,data[7:1]};
                end
                default:;
            endcase
    end

    assign done = (state == stop);
    assign out_byte = data;
    endmodule
  ```
- 带奇偶校验的串口接收器(Serial receiver with parity checking)
  ```Verilog
  // 这题用自己的思路怎么调试都通过不了仿真，人麻了。参考了下一个知乎网友的答案，直接用接受数据的0~7索引值当做状态值，状态转移逻辑的默认情况下索引自增，省去了数据接受状态和计数器，这个状态建模的思路和技巧着实令人惊奇
    module top_module(
        input clk,
        input in,
        input reset,    // Synchronous reset
        output [7:0] out_byte,
        output done
    ); //
        
        // Modify FSM and datapath from Fsm_serialdata
        parameter START = 0, CHECK = 9, STOP = 10, IDEL = 11, ERROR = 12;
        reg [4:0] state, next_state;
        reg valid, start;

        always @(*) begin
            start = 0;
            case (state)
                IDEL: begin next_state = in ? IDEL : START; start = 1; end
                STOP: begin next_state = in ? IDEL : START; start = 1; end
                CHECK: next_state = in ? STOP : ERROR;
                ERROR: next_state = in ? IDEL : ERROR;
                default: next_state = state + 1;
            endcase  
        end
        
        always @(posedge clk)
            if (reset)
                state <= IDEL;
            else begin
                state <= next_state;
                valid <= odd;
            end
        
        always @(posedge clk)
            if ((0 <= state) & (state < 8))
                out_byte[state] <= in;
        
        wire odd;
        parity check(clk, reset | start, in, odd);
        
        assign done = (valid & (state == STOP));
        // New: Add parity checking.
        
    endmodule
  ```
- 序列识别(Sequence recognition)
  ```Verilog
  //
    module top_module(
        input clk,
        input reset,    // Synchronous reset
        input in,
        output disc,
        output flag,
        output err);

        //in为0时进入序列1的计数状态，再次检测到in为0时结束序列1的计数状态
        //若计数器值为5，则输出disc
        //若计数器值为6，则输出flag
        //若计数器值为7，则输出err
        //复位信号将会清除计数器并将所有输出信号清零

        int cnt;
        parameter head = 0, counting = 1, tail = 2, idle = 3;
        reg [1:0] state, next_state;

        always @(*) begin
            case (state)
                idle: 
                    next_state = in ? idle : head;

                head: 
                    next_state = counting;

                counting: 
                    next_state = in ? counting : tail;

                tail: 
                    next_state = (cnt >= 5) ? tail : idle;
            endcase
        end

        always @(posedge clk) begin
            if (reset)
                state <= idle;
            else 
                state <= next_state;
        end

        always @(posedge clk ) begin
            if(reset)
                cnt <= 0;
            else 
                cnt <= (state == counting) ? cnt + 1 : 
                (state == tail ? cnt : 0);
        end

        assign disc = (state == tail && cnt == 5);
        assign flag = (state == tail && cnt == 6);
        assign err = (state == tail && cnt >= 7);

    endmodule
  ```
- Q8:设计米利状态机
  ```Verilog
  //

  ```
- Q5a:摩尔状态机的串口比较器(Serial two's complemetor(Moore FSM))
  ```Verilog
  //

  ```
- Q5b:米利状态机的串口比较器(Serial two's complemetor(Moore FSM))
  ```Verilog
  //

  ```
- Q3a:状态机(FSM)
  ```Verilog
  //

  ```
- Q3b:状态机(FSM)
  ```Verilog
  //

  ```
- Q3c:状态机逻辑(FSM logic)
  ```Verilog
  //

  ```
- Q6b:状态机迭代逻辑(FSM next-state logic)
  ```Verilog
  //

  ```
- Q6c:独热码状态机迭代逻辑(FSM one-hot next-state logic)
  ```Verilog
  //

  ```
- Q6:状态机(FSM)
  ```Verilog
  //

  ```
- Q2a:状态机(FSM)
  ```Verilog
  //

  ```
- Q2b:状态机(FSM)
  ```Verilog
  //

  ```
- Q2a:状态机(FSM)
  ```Verilog
  //

  ```
- Q2b:另一个状态机(Another FSM)
  ```Verilog
  //

  ```

## 构建更大规模电路(Building larger Circuits)
- J
  ```Verilog
  //

  ```
- J
  ```Verilog
  //

  ```
- J
  ```Verilog
  //

  ```
- J
  ```Verilog
  //

  ```
- J
  ```Verilog
  //

  ```

# 验证:读仿真波形(Verification:Reading Simulations)
## 寻找代码Bug（Finding Bugs in code）
- 二路选择器(MUX)
  ```Verilog
  // 注意位宽匹配
    module top_module (
        input sel,
        input [7:0] a,
        input [7:0] b,
        output [7:0] out  );

        assign out = sel ? a : b;

    endmodule
  ```
- 与非门(NAND)
  ```Verilog
  //调用模块时字典传参，结果注意取反
    module andgate (output out, input a, input b, input c, input d, input e);

        assign out = a & b & c & d & e;

    endmodule

    module top_module (input a, input b, input c, output out);

        // 使用 andgate 实现三输入 NAND 门
        wire temp;
        andgate inst1 (.out(temp), .a(a), .b(b), .c(c), .d(1'b1), .e(1'b1));
        assign out = ~temp;

    endmodule
  ```
- 四路选择器(MUX)
  ```Verilog
  // 原题挖了很多坑，首先是命名混乱和重叠,其次wire线宽未定义，最后是sel选中逻辑，最低层应该共用选择信号sel[0]
    module top_module (
        input [1:0] sel,
        input [7:0] a,
        input [7:0] b,
        input [7:0] c,
        input [7:0] d,
        output [7:0] out  ); //

        wire [7:0] Y0, Y1;
        mux2 mux0 ( sel[0],    a,    b, Y0 );
        mux2 mux1 ( sel[0],    c,    d, Y1 );
        mux2 mux_final ( sel[1], Y0, Y1,  out );

    endmodule
  ```
- 加法减法器(Add/Sub)
  ```Verilog
  // 原题挖了两个坑，一个是out的判零逻辑不能用~out,其次是没有预支匹配的else块产生了latch导致仿真不通过
    module top_module ( 
        input do_sub,
        input [7:0] a,
        input [7:0] b,
        output reg [7:0] out,
        output reg result_is_zero
    );//

        always @(*) begin
            case (do_sub)
            0: out = a+b;
            1: out = a-b;
            endcase

            if (out == 8'b0)
                result_is_zero = 1;
            else
                result_is_zero = 0;
        end

    endmodule
  ```
- case声明(case statement)
  ```Verilog
  // 原题留了两个大坑:1 valid和out的case块没有完备; 2 在key3和key9那里进制和位长动了手脚。慢慢看波形的mismatch排查出问题
    module top_module (
        input [7:0] code,
        output reg [3:0] out,
        output reg valid=1 );//

        always @(*)
            case (code)
                8'h45: begin out = 0; valid = 1; end
                8'h16: begin out = 1; valid = 1; end
                8'h1e: begin out = 2; valid = 1; end
                8'h26: begin out = 3; valid = 1; end
                8'h25: begin out = 4; valid = 1; end
                8'h2e: begin out = 5; valid = 1; end
                8'h36: begin out = 6; valid = 1; end
                8'h3d: begin out = 7; valid = 1; end
                8'h3e: begin out = 8; valid = 1; end
                8'h46: begin out = 9; valid = 1; end
                default: begin
                    valid = 0;
                    out = 0;
                end 
            endcase

    endmodule
  ```
## 从仿真波形中构建电路(Build a circuit from simulation waveform)
- 组合逻辑电路1()
    ```Verilog
    // 送分题不多解释了
    module top_module (
        input a,
        input b,
        output q );//

        assign q = a & b; // Fix me

    endmodule
    ```
- 组合逻辑电路2()
  ```Verilog
  // 我这懒狗无法容忍四变量的卡诺图化简，看波形规律不难发现这是个四输入偶数校验电路，四路输入中有偶数个1(0个也算偶数)，输出就为1
    module top_module (
        input a,
        input b,
        input c,
        input d,
        output q );//

        assign q = ~(a ^ b ^ c ^ d); // Fix me

    endmodule
  ```
- 组合逻辑电路3()
  ```Verilog
  // 看图找规律，这题波形可以看得出q凸起的部分，cd输入波形都是有共性的，然而有共性的第一段却没凸起，此时观察ab发现都是低电平，凸起的部分ab至少有一个高电平。再分析cd凸起部分的共性，cd至少一个是高电平，至此逻辑完成。
    module top_module (
        input a,
        input b,
        input c,
        input d,
        output q );//

        assign q = (a | b) & (c | d);

    endmodule
  ```
- 组合逻辑电路4()
  ```Verilog
  // 这题直接看输出波形的凸起部分不容易看出端倪，换个视角看凹陷部分，仅仅在bc都为低电平时q才为低电平
    module top_module (
        input a,
        input b,
        input c,
        input d,
        output q );//

        assign q = b | c; // Fix me

    endmodule
  ```
- 组合逻辑电路5()
  ```Verilog
  // 看了好一会原来是个选择器电路
    module top_module (
        input [3:0] a,
        input [3:0] b,
        input [3:0] c,
        input [3:0] d,
        input [3:0] e,
        output [3:0] q );

        always @(*) begin
            case(c)
                4'b0000: q = b;
                4'b0001: q = e;
                4'b0010: q = a;
                4'b0011: q = d;
                default: q = 4'b1111; // Default case to handle unexpected values
            endcase
        end

    endmodule
  ```
- 组合逻辑电路6()
  ```Verilog
  // 又中了出题者的空城计，还以为有什么映射规律，仍然是个case语句，然而这题踩了个大坑就是16进制位长我声明为4了导致结果怎么都不对，最后是在VSCode的Verilog语法检查器下才发觉的。
    module top_module (
        input [2:0] a,
        output [15:0] q ); 

        always @(*) begin
            case(a)
                3'b000: q = 16'h1232;
                3'b001: q = 16'haee0;
                3'b010: q = 16'h27d4;
                3'b011: q = 16'h5a0e;
                3'b100: q = 16'h2066;
                3'b101: q = 16'h64ce;
                3'b110: q = 16'hc526;
                3'b111: q = 16'h2f19;
                default: q = 16'h0000;
            endcase
        end

    endmodule
  ```
- 时序逻辑电路7()
  ```Verilog
  // 送分题
    module top_module (
        input clk,
        input a,
        output q );

        always @(posedge clk ) begin
            q = ~a;
        end

    endmodule
  ```
- 时序逻辑电路8()
  ```Verilog
  // p高电平保持，低电平为a。这题又困了好一会，因为q的逻辑误解为了每逢下降沿则翻转导致q仿真波形一直是相反的，还在纠结q的赋初值问题，实际q的赋值逻辑是下降沿为p。
    module top_module (
        input clock,
        input a,
        output p,
        output q );

        always @(negedge clock) begin
            q <= p;
        end

        assign p = (clock) ? a : p;

    endmodule
  ```
- 时序逻辑电路9()
  ```Verilog
  // 带同步复位的7进制计数器逻辑
    module top_module (
        input clk,
        input a,
        output [3:0] q );

        always @(posedge clk) begin
            if(a)
                q <= 4;
            else
                q <= (q == 6) ? 0 : q + 1;
        end

    endmodule
  ```
- 时序逻辑电路10()
  ```Verilog
  // 这题观察q的波形很容易看出state决定着输出同或还是异或，然而state的迭代规则用自己的卡诺图错位波形怎么都得不出正确结果，实在忍不了了还是参考网友答案的。
    module top_module (
        input clk,
        input a,
        input b,
        output q,
        output state  );

        always @(posedge clk ) begin
            if(a & b) state <= 1;
            else if(~a & ~b) state <= 0;
            else state <= state;
        end
        assign q = state ? ~(a ^ b) : (a ^ b);
    endmodule

  ```
# 验证:编写TestBench
- 时钟(Clock)
  ```verilog
  // 这题注意always和forever生成时钟脉冲的区别，initial块内尽量用forever，always虽也能用但更常用的还是设计综合语法居多。
    module top_module ( );
        reg clk;
        
        dut test(.clk (clk));
        initial begin
            clk = 0;
            forever #5 clk = ~clk;
        end
    endmodule
  ```
- TestBench1
  ```verilog
  // 这题坑点是不熟悉Verilog仿真语法,#后接的数字是相对上一次#后经过的时间，而不是绝对时间轴时间
  module top_module ( output reg A, output reg B );//

    // generate input patterns here
    initial begin
		A = 0; B = 0;
        #10 A=1;
        #5 B=1;
        #5 A=0;
        #20 B=0;
    end

    endmodule
  ```
- 与门(AND gate)
  ```verilog
  // 这题只管给输入in合适的激励信号，至于输出Out是怎样的不用管，另外不要在仿真末尾调用$finish不然会提前结束仿真通不过。
    module top_module();
        reg [1:0] in;
        reg out;

        andgate test(.in (in), .out (out));
        initial begin
            in = 2'b00; 
            #10 in = 2'b01; 
            #10 in = 2'b10; 
            #10 in = 2'b11;
        end
    endmodule
  ```
- TestBench2()
  ```verilog
  // 这题又踩坑了，把时钟clk的仿真信号和输入的仿真信号放在一个initial块内，导致forever块后的仿真一直未能执行。
    module top_module();
        reg clk;
        reg in;
        reg [2:0] s;
        reg out;

        q7 test(.clk(clk), .in(in), .s(s), .out(out));

        initial begin
            clk = 0; 
            forever #5 clk = ~clk;
        end
        
        initial begin
            in = 0; s = 3'd2;
            #10 s = 3'd6;
            #10 s = 3'd2; in = 1;
            #10 s = 3'd7; in = 0;
            #10 s = 3'd0; in = 1;
            #30 in = 0;
        end
    endmodule
  ```
- T触发器(T flip-flop)
  ```verilog
  // 由于对Verilog仿真语法的不熟悉，不知道仿真信号什么时候通过initial块生成，什么时候也可以用时序逻辑生成，这题还是参考网友答案的有点懵，虽然simulation部分写完了但基本踩了很多坑参考了网友答案的，得好好补补这部分的语法知识。
    module top_module ();
        reg clk;
        reg reset;
        reg t;
        reg q;

        tff test(.clk(clk), .reset(reset), .t(t), .q(q));

        initial begin
            clk = 0;
            forever #5 clk = ~clk;
        end

        initial begin
            reset = 0;
            #10 reset = 1;
            #10 reset = 0;
        end

        always @(posedge clk) begin
            if(reset)
                t <= 0;
            else
                t <= 1;
        end
    endmodule
  ```
# CS450
- 定时器(timer)
  ```verilog
  // 因为没捋清count的自减逻辑把自减逻辑写在另一个if else块了，看似简单的需求并没有一遍过
    module top_module(
        input clk, 
        input load, 
        input [9:0] data, 
        output tc
    );
        reg [9:0] count;

        always @(posedge clk ) begin
            if(load)
                count <= data;
            else
                if(count != 10'b0)
                    count <= count - 1;
        end

        assign tc = (count == 10'b0);

    endmodule
  ```
- 饱和计数器(counter 2bc)
  ```verilog
  // 看题干背景分支预测一堆描述，本以为要用状态机的，结果就是一饱和计数器逻辑
    module top_module(
        input clk,
        input areset,
        input train_valid,
        input train_taken,
        output [1:0] state
    );
        reg[1:0] count;


        always @(posedge clk or posedge areset) begin
            if(areset)
                count <= 2'b01;
            else
                casez({train_valid, train_taken})
                    2'b0x: count <= count; 
                    2'b10: count <= (count == 3'b00) ? count : count - 1;
                    2'b11: count <= (count == 3'b11) ? count : count + 1;
                endcase
        end

        assign state = count;

    endmodule
  ```
- 历史移位寄存器(history shift)
  ```verilog
  // 这题最大的难度是读懂题意。
    module top_module(
        input clk,
        input areset,

        input predict_valid,
        input predict_taken,
        output [31:0] predict_history,

        input train_mispredicted,
        input train_taken,
        input [31:0] train_history
    );

        always @(posedge clk , posedge areset) begin
            if(areset)
                predict_history <= 32'b0;
            else
                if(train_mispredicted) 
                    predict_history <= {train_history[30:0], train_taken};
                else if(predict_valid)
                    predict_history <= {predict_history[30:0], predict_taken};
                else
                    predict_history <= predict_history;
        end

    endmodule
  ```
- gshare分支预测(gshare predictor)
  ```verilog
  // 实在过不了仿真也看不懂题意耻辱收尾，还是参考了网友答案，看题解貌似是综合运用了前几题的相关逻辑，怪不着这个系列题没更新下去了，感觉没点计算机体系结构的前置知识还是不好做的
    module top_module(
        input clk,
        input areset,

        input  predict_valid,
        input  [6:0] predict_pc,
        output predict_taken,
        output [6:0] predict_history,

        input train_valid,
        input train_taken,
        input train_mispredicted,
        input [6:0] train_history,
        input [6:0] train_pc
    );
        
        reg[1:0] PHT[127:0];
        
        always@(posedge clk or posedge areset)
            begin
                if(areset)
                    begin
                        predict_history<=0;
                        for(int i=0;i<128;i++)
                            PHT[i]<=2'b01;
                    end
                else
                    begin
                        if(train_valid && train_mispredicted)
                            predict_history<={train_history[6:0],train_taken};
                        else if(predict_valid)
                            predict_history<={predict_history[6:0],predict_taken};
                        
                        if(train_valid)
                            begin
                                if(train_taken)
                                    PHT[train_history^train_pc]<=(PHT[train_history^train_pc]==2'b11)?2'b11:(PHT[train_history^train_pc]+1);
                                else	
                                    PHT[train_history^train_pc]<=(PHT[train_history^train_pc]==2'b00)?2'b00:(PHT[train_history^train_pc]-1);
                            end
                    end
            end
        
        assign predict_taken = PHT[predict_history^predict_pc][1];

    endmodule
  ```