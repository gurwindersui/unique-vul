DoS Attack on Liquidity Withdrawals During Market Volatility#163 Status



Summary Attackers can exploit market volatility periods to perform cost-effective partial DoS attacks, preventing liquidity providers from withdrawing funds during market crashes. This results in significant financial losses as users cannot exit positions at optimal prices and may face forced liquidations.

Finding Description The protocol's liquidity supply mechanism contains a critical vulnerability that allows sophisticated attackers to time DoS attacks during high-volatility market events, specifically targeting the withdrawal functionality when users most need liquidity access.

Attack Vector:)

Users supply liquidity through the supply function using APTOS and SUI tokens.

```rust
public entry fun supply(
    account: &signer,
    asset: address,
    amount: u256,
    on_behalf_of: address,
    referral_code: u16
)
```

The system validates reserves are not paused/frozen

```rust
assert!(!is_paused, error_config::get_ereserve_paused());
assert!(!is_frozen, error_config::get_ereserve_frozen());
```

During market crashes, when users panic and attempt mass withdrawals, attackers execute partial DoS attacks Despite Aptos claiming 150k TPS capacity, real-world performance shows:)

Peak: 32k TPS Average: 12k TPS it is not highly cost expensive.

Users cannot access the withdraw function during critical market periods

```rust
public entry fun withdraw(
    account: &signer,
    asset: address,
    amount: u256,
    to: address
)
```

Market crash triggers user panic → Mass withdrawal attempts → DoS attack execution to withdraw function → Failed withdrawals → Forced holding during price decline → Additional liquidations via

```rust
public entry fun set_user_use_reserve_as_collateral(
    account: &signer,
    asset: address,
    use_as_collateral: bool
) {
    validation_logic::validate_hf_and_ltv(
        &user_config_map,
        asset,
        account_address,
        reserves_count,
        user_emode_category,
        emode_ltv,
        emode_liq_threshold
    );
}
```

```rust
public fun validate_health_factor(
    user_config_map: &UserConfigurationMap,
    user: address,
    user_emode_category: u8,
    reserves_count: u256,
    emode_ltv: u256,
    emode_liq_threshold: u256
): (u256, bool) {
    let (_, _, _, _, health_factor, has_zero_ltv_collateral) =
        generic_logic::calculate_user_account_data(
            user_config_map,
            reserves_count,
            user,
            user_emode_category,
            emode_ltv,
            emode_liq_threshold
        );

    assert!(
        health_factor >= user_config::get_health_factor_liquidation_threshold(),
        error_config::get_ehealth_factor_lower_than_liquidation_threshold()
    );

    (health_factor, has_zero_ltv_collateral)
}
```

when threshold goes below 1e18 (its divergent Loss to another users). Pre-crash APTOS price: $17.00 USD

Target withdrawal price: $13.10-13.99 USD

Actual withdrawal price (post-DoS): $5.50 USD

User loss: ~67-70% of portfolio value

Capital Requirements: Moderate (partial DoS vs. full DoS)

Timing Requirements: Predictable (market volatility events)

Success Probability: High during volatile periods

Medium Severity classification justified under Cantina rules:)

Temporary Disruption or DoS: A bug that leads to temporary downtime or a denial of service (DoS). This may cause users to experience disruptions, but doesn’t necessarily compromise the security of the protocol.

Issues with significant constraints, such as capital requirement, previous planning, or actions by other users

Impact Explanation Liquidity providers unable to exit positions

Arbitrage traders are still in loss

MEV bots are still in loss

Users facing forced liquidations due to health_factor threshold breaches less than 1e18. (divergent Loss)

Likelihood Explanation Crypto market crashes are cyclical and identifiable:)

Real TPS limitations make DoS attacks feasible and cost-effective:) with advanced move scripts.

Attackers can profit from maintained positions during others' forced liquidations

Partial DoS requires significantly less capital than full network DoS.

Panic selling during crashes creates predictable mass withdrawal patterns:)

Proof of Concept simple demonstration:) liquidity Provider deposits: 1,000 APTOS tokens

Initial APTOS price: $17.00 USD

Initial investment value: $17,000 USD

T=0 (Normal Market):)

APTOS price: $17.00

User portfolio: 1,000 APTOS = $17,000

Network TPS: ~1-2k (normal operation)

T+1 Hour (Market Crash Begins):(

APTOS price drops to: $13.50 (-20.6%)

User attempts withdrawal: 1,000 APTOS = $13,500

Optimal exit value: $13,500 ✓ (User should withdraw here)

T+1 Hour (DoS Attack Initiated):(

Attacker executes partial DoS on withdrawal functions

Network congestion: 32k+ TPS requests overwhelm actual 1-2k capacity

Users cannot execute public entry fun withdraw function

T+3 Hours (Continued Price Decline):

APTOS price: $9.80 (-42.4%)

DoS continues, withdrawals still blocked

User portfolio value: $9,800

T+6 Hours (Peak Panic Phase)

APTOS price: $5.50 (-67.6%)

Users with health_factor < 1e18 get automatically liquidate

Mass liquidations

T+8 Hours (DoS Ends):)

Attacker releases DoS attack

APTOS price stabilizes at: $5.50

User finally able to withdraw: 1,000 APTOS = $5,500

Total Loss Calculation:

Intended exit value: $13,500

Actual exit value: $5,500

Direct loss: $8,000 (47.1% of original investment)

Opportunity cost: $13,500 - $5,500 = $8,000

Total effective loss: $11,500 (67.6% of original investment)

Monitor market conditions for volatility indicators Position attack infrastructure during normal market conditions Detect market crash initiation (price drop >10% in <1 hour) Execute partial DoS attack targeting withdrawal functions Maintain attack during peak panic withdrawal period (2-6 hours) Release DoS after optimal profit extraction during Divergent Loss.
