module top_module(
    input  clk,
    input  reset,
    input  in,       
    output [3:0] an,
    output [7:0] ca,
    output buzzer
);
    wire [15:0] pulse;
    wire [31:0] rpm;

    RPM_measure rpm_m (
        .clk(clk),
        .reset(reset),
        .in(in),
        .pulse(pulse),
        .rpm(rpm),
        .buzzer(buzzer_signal)
    );
    assign buzzer=buzzer_signal;

    seven_seg display_m (
        .clk(clk),
        .rpm(rpm),
        .an(an),
        .ca(ca)
    );
endmodule


module timer(
    input        clk,
    input        reset,
    output reg   second
);
    reg [31:0] count;

    always @(posedge clk or posedge reset) begin
        if (reset) begin
            count  <= 0;
            second <= 0;
        end else if (count == 50_000_000-1) begin
            count  <= 0;
            second <= 1;
        end else begin
            count  <= count + 1;
            second <= 0;
        end
    end
endmodule


module hall_sensor(
    input  clk,
    input  reset,
    input  in,
    output reg out
);
    reg prev;

    always @(posedge clk or posedge reset) begin
        if (reset) begin
            prev <= 0;
            out  <= 0;
        end else begin
            out  <= in & ~prev;
            prev <= in;
        end
    end
endmodule


module RPM_measure(
    input              clk,
    input              reset,
    input              in,
    output reg [15:0]  pulse,
    output reg [31:0]  rpm,
    output buzzer
);
    wire        hall_edge;
    wire        second;
    reg buzzer_reg=0;

    hall_sensor hs (
        .clk(clk),
        .reset(reset),
        .in(in),
        .out(hall_edge)
    );

    timer tm (
        .clk(clk),
        .reset(reset),
        .second(second)
    );

    always @(posedge clk or posedge reset) begin
        if (reset) begin
            pulse <= 0;
            rpm   <= 0;
            buzzer_reg<=0;
        end else begin
            if (hall_edge) begin
                pulse <= pulse + 1;
                if(pulse*60>500)
                    buzzer_reg<=1;
                else
                    buzzer_reg<=0;
            end
            if (second) begin
                rpm   <= pulse * 60;
                pulse <= 0;
            end
        end
    end
endmodule


module seven_seg(
    input        clk,
    input [31:0] rpm,
    output reg [3:0] an,
    output reg [7:0] ca
);
    reg        clk_seg = 0;
    reg [31:0] count   = 0;
    reg  [1:0] sel     = 0;

    always @(posedge clk) begin
        if (count == 100_000-1) begin
            count   <= 0;
            clk_seg <= ~clk_seg;
        end else
            count <= count + 1;
    end

    reg [3:0] digit [3:0];
    always @(posedge clk) begin
        digit[3] <= rpm / 1000;
        digit[2] <= (rpm % 1000) / 100;
        digit[1] <= (rpm % 100) / 10;
        digit[0] <= rpm % 10;
    end

    wire [7:0] seg [3:0];
    display d0(.clk(clk), .value(digit[0]), .ca(seg[0]));
    display d1(.clk(clk), .value(digit[1]), .ca(seg[1]));
    display d2(.clk(clk), .value(digit[2]), .ca(seg[2]));
    display d3(.clk(clk), .value(digit[3]), .ca(seg[3]));

    always @(posedge clk_seg) begin
        case (sel)
            2'b00: begin an <= 4'b0001; ca <= seg[3]; end
            2'b01: begin an <= 4'b0010; ca <= seg[2]; end
            2'b10: begin an <= 4'b0100; ca <= seg[1]; end
            2'b11: begin an <= 4'b1000; ca <= seg[0]; end
        endcase
        
        
        sel <= sel + 1;
    end
endmodule


module display(
    input        clk,
    input  [3:0] value,
    output reg [7:0] ca
);
    always @(posedge clk) begin
        case (value)
            4'd0: ca <= 8'b11000000;
            4'd1: ca <= 8'b11111001;
            4'd2: ca <= 8'b10100100;
            4'd3: ca <= 8'b10110000;
            4'd4: ca <= 8'b10011001;
            4'd5: ca <= 8'b10010010;
            4'd6: ca <= 8'b10000010;
            4'd7: ca <= 8'b11111000;
            4'd8: ca <= 8'b10000000;
            4'd9: ca <= 8'b10010000;
        endcase
    end
endmodule
