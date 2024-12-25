# 디지털 시스템 설계

본 프로젝트는 Verilog를 이용하여 LED의 상태를 제어하는 FSM(Finite State Machine)을 설계하고, 이를 실제 보드에서 구현하는 과정에 대해 다룹니다.

## 목차
1. [프로젝트 개요](#프로젝트-개요)
2. [LED 동작](#led-동작)
3. [FSM 설계](#fsm-설계)
4. [Verilog 소스코드](#verilog-소스코드)
5. [보드 동작 사진](#보드-동작-사진)

---

## 프로젝트 개요
이 프로젝트에서는 키보드 입력에 따라 LED의 상태를 변화시키는 디지털 시스템을 설계했습니다. FSM(Finite State Machine)을 기반으로 설계하였으며, Verilog HDL을 사용하여 구현했습니다.

---

## LED 동작
키보드에서 특정 키를 입력하면 LED가 다음과 같은 동작을 수행합니다.

- `s` 키: LED 상태가 순차적으로 변환
- `r` 키: LED 상태가 리셋
- `t` 키: LED 상태가 증가
- `b` 키: LED 상태가 감소

![LED 동작](path/to/diagram.png)
![LED 동작](path/to/diagram.png)
![LED 동작](path/to/diagram.png)
![LED 동작](path/to/diagram.png)
![LED 동작](path/to/diagram.png)
![LED 동작](path/to/diagram.png)
![LED 동작](path/to/diagram.png)
![LED 동작](path/to/diagram.png)

---

## FSM 설계
### 아이디어 설명
Lab 3의 FSM을 기반으로 키보드 입력에 따라 동작하도록 설계되었습니다.

- 특정 키 입력(`s`, `r`, `t`, `b`)을 인식하여 FSM의 상태를 변화시킴.
- 상태 전환 규칙:
  - 한 방향으로 증가하거나 감소
  - 특정 간격(3씩)으로 증가 및 감소하는 동작 추가
- FSM 설계 시 `case` 문과 상태 인코딩을 활용하여 간결하고 직관적인 동작을 구현.

---

## Verilog 소스코드
다음은 본 프로젝트에서 사용된 Verilog 소스코드입니다.

```verilog
module roulette(clk, rst, ps2c, ps2d, y);
    input wire clk, rst;
    input wire ps2c, ps2d;
    output wire [7:0] y;

    localparam BRK = 8'hf0;
    localparam T = 8'h2c;
    localparam B = 8'h32;
    localparam S = 8'h1b;
    localparam R = 8'h2d;

    wire [7:0] scan_out;
    reg [7:0] y;
    localparam s0=0, s1=1, s2=2, s3=3, s4=4, s5=5, s6=6, s7=7;
    reg [2:0] state, nstate;
    wire scan_done_tick;

    ps2_rx ps2_rx_unit (
        .clk(clk), .rst(rst), .rx_en(1'b1),
        .ps2d(ps2d), .ps2c(ps2c),
        .rx_done_tick(scan_done_tick), .dout(scan_out)
    );

    always @ (posedge clk, posedge rst) begin
        if (rst == 1'b1) state <= s0;
        else state <= nstate;
    end

    always @ (*) begin
        y = 0;
        nstate = state;

        case (state)
            s0: begin
                if (scan_done_tick && scan_out == S) nstate = s1;
                else if (scan_done_tick && scan_out == B) nstate = s5;
                else if (scan_done_tick && scan_out == T) nstate = s3;
                else if (scan_done_tick && scan_out == R) nstate = s4;
                y[0] = 1'b1;
            end
            // 나머지 상태 정의 생략...
        endcase
    end
endmodule
