# oxide-mermaid-theme

A Mermaid theme built from Oxide's dark-green terminal aesthetic.

Inspiration comes from this cool looking diagram in [RFD 0363](https://rfd.shared.oxide.computer/rfd/0363):

[![Top Level Block Diagram](top_level_block_diagram.png)](https://rfd.shared.oxide.computer/rfd/image/363/figures/top_level_block_diagram.png)

## Mermaid demo

### Flowchart

Minibar Manufacturing Tester.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'background': '#1c2225', 'primaryColor': '#1c372e', 'primaryTextColor': '#48d597', 'primaryBorderColor': '#48d597', 'secondaryColor': '#1c372e', 'tertiaryColor': '#1c2225', 'lineColor': '#48d597', 'fontFamily': 'Iosevka Nerd Font Mono', 'titleColor': '#48d597', 'noteBkgColor': '#1c2225', 'noteTextColor': '#48d597', 'noteBorderColor': '#034e3a'}, 'flowchart': {'curve': 'stepBefore'}}}%%
graph LR
    subgraph Sled_Interfaces [Sled Interfaces]
        direction TB
        Sled_Power_Connector[Sled Power Connector]
        ExaMax_Connector1[ExaMax Connector]
        ExaMax_Connector2[ExaMax Connector]
        ExaMax_Connector3[ExaMax Connector]
    end

    subgraph Bench_Interfaces [Bench & Management Interfaces]
        direction TB
        Bench_Power_Connector[Bench Power Connector]
        PCIe_Connector[PCIe Gen4 x16 CEM Connector]
        Minibar_SP[Ethernet Jack]
        Sled_SP_Link_0[Ethernet Jack]
        Sled_SP_Link_1[Ethernet Jack]
    end

    Hot_Swap_Controller1[Hot Swap Controller]
    Hot_Swap_Controller2[Hot Swap Controller]

    Bench_Power_Connector --> Hot_Swap_Controller1
    Hot_Swap_Controller1 -- "54.5V" --> Sled_Power_Connector
    Bench_Power_Connector --> Hot_Swap_Controller2

    Minibar_System_Power_Rails[Minibar System Power Rails]
    Hot_Swap_Controller2 -- "54.5V" --> Minibar_System_Power_Rails

    eFuse1[eFuse]
    eFuse2[eFuse]

    Minibar_System_Power_Rails -- "12V" --> eFuse1
    Minibar_System_Power_Rails -- "3.3V" --> eFuse2

    eFuse1 -- "12V" --> PCIe_Connector
    eFuse2 -- "3.3V" --> PCIe_Connector

    Sled_Power --- ExaMax_Connector1
    ExaMax_Connector1 <-- "PCIe Gen3/Gen4 x4" --> PCIe_Connector

    Clock_Buffer[Clock Buffer]
    ExaMax_Connector1 -- "PCIe REFCLK" --> Clock_Buffer
    Clock_Buffer -- "PCIe REFCLK" --> PCIe_Connector

    Lattice_ECP5_FPGA[Lattice ECP5 FPGA]

    ExaMax_Connector1 -- "PCIe Aux Signals" --> Lattice_ECP5_FPGA
    Lattice_ECP5_FPGA -- "Output Enable/Config" --> Clock_Buffer
    Lattice_ECP5_FPGA <-- "PCIe Aux Signals" --> PCIe_Connector

    Sled_Power --- ExaMax_Connector2
    ExaMax_Connector2 <-- "Ignition LVDS" --> Lattice_ECP5_FPGA

    VSC7448_Switch[VSC7448 Switch]
    ExaMax_Connector2 <-- "SGMII" --> VSC7448_Switch

    Sled_Power --- ExaMax_Connector3
    ExaMax_Connector3 <-- "200G/100GBASE-KR4" --> ExaMax_Connector2
    ExaMax_Connector3 <-- "Ignition LVDS" --> Lattice_ECP5_FPGA
    ExaMax_Connector3 <-- "SGMII" --> VSC7448_Switch

    STM32H753[STM32H753 Service Processor]
    Lattice_ECP5_FPGA <-- "SPI" --> STM32H753
    Clock_Buffer <-- "I2C" --> STM32H753

    LPC55S6x[LPC55S6x RoT]
    LPC55S6x <-- "SWD" --> STM32H753
    LPC55S6x <-- "SPI" --> STM32H753

    Aux_Flash[Aux Flash]
    Aux_Flash <-- "QUADSPI" --> STM32H753

    VPD_EEPROM[VPD EEPROM]
    VPD_EEPROM <-- "I2C" --> STM32H753

    KSZ8463_Switch[KSZ8463 Switch]
    STM32H753 <-- "RMII" --> KSZ8463_Switch
    STM32H753 <-- "SPI" --> KSZ8463_Switch

    VSC8562[VSC8562 PHY]
    VSC8562 <-- "MIIM" --> STM32H753
    KSZ8463_Switch <-- "100BASE-FX" --> VSC8562
    KSZ8463_Switch <-- "100BASE-FX" --> VSC8562

    VSC8562 <-- "SGMII" --> VSC7448_Switch
    VSC8562 <-- "SGMII" --> VSC7448_Switch

    VSP8562 <-- "BASE-T" --> Minibar_SP
    VSP8562 <-- "BASE-T" --> Sled_SP_Link_0
    VSP8562 <-- "BASE-T" --> Sled_SP_Link_1
```

### Sequence diagram

From [RFD 0373](https://rfd.shared.oxide.computer/rfd/0373#_target_driven).

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'background': '#1c2225', 'primaryColor': '#1c372e', 'primaryTextColor': '#48d597', 'primaryBorderColor': '#48d597', 'secondaryColor': '#1c372e', 'tertiaryColor': '#1c2225', 'lineColor': '#48d597', 'fontFamily': 'Iosevka Nerd Font Mono', 'titleColor': '#48d597', 'noteBkgColor': '#1c2225', 'noteTextColor': '#48d597', 'noteBorderColor': '#034e3a'}, 'flowchart': {'curve': 'stepBefore'}}}%%
sequenceDiagram
    participant External client
    participant Nexus
    participant CockroachDB
    participant Target

    note over Nexus: API request to change configuration

    External client ->> Nexus: reconfigure
    activate Nexus
    Nexus ->> CockroachDB: create new generation N for config
    activate CockroachDB
    CockroachDB -->> Nexus: ok
    deactivate CockroachDB
    Nexus -->> External client: ok
    deactivate Nexus

    note over Nexus: Activate RPW (Nexus side)
    activate Nexus
    Nexus ->> CockroachDB: list targets
    activate CockroachDB
    CockroachDB -->> Nexus: list of targets
    deactivate CockroachDB

    Nexus ->> CockroachDB: fetch intended state
    activate CockroachDB
    CockroachDB -->> Nexus: state generation N
    deactivate CockroachDB

    Nexus ->> Target: you need to update
    activate Target
    Target -->> Nexus: ok

    deactivate Nexus
    note over Nexus: End of RPW (Nexus side)

    note over Target: Activate RPW (Target side)
    activate Target

    Target ->> Nexus: which generation should I have?
    activate Nexus
    Nexus ->> CockroachDB: fetch intended state
    activate CockroachDB
    CockroachDB -->> Nexus: generation N
    deactivate CockroachDB
    Nexus -->> Target: generation N (includes contents)
    deactivate Nexus

    deactivate Target
    note over Target: apply generation N
```