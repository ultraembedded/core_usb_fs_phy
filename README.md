### USB Full Speed PHY

Github:   [http://github.com/ultraembedded/core_usb_fs_phy](https://github.com/ultraembedded/core_usb_fs_phy)

This component implements the low level USB1.1 / FS / 12Mbit/s USB signalling (SOP,DATA,EOP) with the required bitstuffing.

##### Features
* UTMI PHY interface.
* Connection to a transceiver (e.g. USB1T11A) or can drive the FPGA pins directly as USB D+/D-.

##### Configuration / Requirements
* Top: usb_fs_phy
* Clock: clk_i - 48MHz
* Reset: rst_i - Asynchronous, active high

##### Testing
Tested with various FPGA + USB based projects.

##### FPGA Transceiver
To avoid external components, you can drive the USB D+/D- pins directly from the FPGA fabric.
The USB1.1 bit rate is low enough (12MHz) for this to be not too problematic with short cables!

```
module usb_fs_phy_wrapper
(
     input           clk_i
    ,input           rst_i

    ,input  [  7:0]  utmi_data_out_i
    ,input           utmi_txvalid_i
    ,input  [  1:0]  utmi_op_mode_i
    ,input  [  1:0]  utmi_xcvrselect_i
    ,input           utmi_termselect_i
    ,input           utmi_dppulldown_i
    ,input           utmi_dmpulldown_i
    ,output [  7:0]  utmi_data_in_o
    ,output          utmi_txready_o
    ,output          utmi_rxvalid_o
    ,output          utmi_rxactive_o
    ,output          utmi_rxerror_o
    ,output [  1:0]  utmi_linestate_o

    // USB D+ / D-
    ,inout          usb_dp_io
    ,inout          usb_dn_io
);

wire           usb_pads_rx_rcv_w;
wire           usb_pads_rx_dn_w;
wire           usb_pads_rx_dp_w;
wire           usb_pads_tx_dn_w;
wire           usb_pads_tx_dp_w;
wire           usb_pads_tx_oen_w;

usb_transceiver u_usb_xcvr
(
    // Inputs
     .usb_phy_tx_dp_i(usb_pads_tx_dp_w)
    ,.usb_phy_tx_dn_i(usb_pads_tx_dn_w)
    ,.usb_phy_tx_oen_i(usb_pads_tx_oen_w)
    ,.mode_i(1'b1)

    // Outputs
    ,.usb_dp_io(usb_dp_io)
    ,.usb_dn_io(usb_dn_io)
    ,.usb_phy_rx_rcv_o(usb_pads_rx_rcv_w)
    ,.usb_phy_rx_dp_o(usb_pads_rx_dp_w)
    ,.usb_phy_rx_dn_o(usb_pads_rx_dn_w)
);

usb_fs_phy u_usb_phy
(
    // Inputs
     .clk_i(clk_i)
    ,.rst_i(rst_i)
    ,.utmi_data_out_i(utmi_data_out_i)
    ,.utmi_txvalid_i(utmi_txvalid_i)
    ,.utmi_op_mode_i(utmi_op_mode_i)
    ,.utmi_xcvrselect_i(utmi_xcvrselect_i)
    ,.utmi_termselect_i(utmi_termselect_i)
    ,.utmi_dppulldown_i(utmi_dppulldown_i)
    ,.utmi_dmpulldown_i(utmi_dmpulldown_i)
    ,.usb_rx_rcv_i(usb_pads_rx_rcv_w)
    ,.usb_rx_dp_i(usb_pads_rx_dp_w)
    ,.usb_rx_dn_i(usb_pads_rx_dn_w)
    ,.usb_reset_assert_i(1'b0)

    // Outputs
    ,.utmi_data_in_o(utmi_data_in_o)
    ,.utmi_txready_o(utmi_txready_o)
    ,.utmi_rxvalid_o(utmi_rxvalid_o)
    ,.utmi_rxactive_o(utmi_rxactive_o)
    ,.utmi_rxerror_o(utmi_rxerror_o)
    ,.utmi_linestate_o(utmi_linestate_o)
    ,.usb_tx_dp_o(usb_pads_tx_dp_w)
    ,.usb_tx_dn_o(usb_pads_tx_dn_w)
    ,.usb_tx_oen_o(usb_pads_tx_oen_w)
    ,.usb_reset_detect_o()
    ,.usb_en_o()
);

endmodule
```

Example contraints;
```
set_property -dict { PACKAGE_PIN N4    IOSTANDARD LVCMOS33 } [get_ports { usb_dp_io }];
set_property -dict { PACKAGE_PIN P3    IOSTANDARD LVCMOS33 } [get_ports { usb_dn_io }];
set_property PULLUP TRUE [get_ports usb_dp_io]
```

##### References
* [UTMI Specification](https://www.intel.com/content/dam/www/public/us/en/documents/technical-specifications/usb2-transceiver-macrocell-interface-specification.pdf)
* [USB1T11A](http://www.mouser.com/ds/2/149/fairchild%20semiconductor_usb1t11a-320893.pdf)
