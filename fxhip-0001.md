# FXHIP-0001: Pre-Bonding Tranche Anti-Sniping System

**Type:** Protocol Enhancement  
**Status:** Draft  
**Author:** ir8prim8 
**Created:** July 10, 2025  
**Updated:** July 10, 2025

## Abstract

This proposal introduces a Pre-Bonding Tranche system to mitigate sniping attacks on fxhash art coin launches. The system creates a safelist-based early access period where eligible participants can contribute $FXH with equal pricing before the standard bonding curve launches, reducing the concentration of early tokens in the hands of MEV bots and sophisticated snipers.

## Motivation

### Current Problem

The existing bonding curve launch mechanism suffers from systematic sniping attacks that undermine fair price discovery and liquidity provision:

1. **Sniping Prevalence**: Every pool launched has been sniped to some degree. Most recently, GALO experienced as much as 1/3 of initial liquidity drained in the first 8 hours
2. **Concentrated Extraction**: Single actors can extract substantial portions of the curve's bottom half, increasing risk to other participants
3. **Ineffective Existing Mitigations**: Initial curve adjustments implemented for GALO did not effectively address the sniping problem
4. **Legitimate Collector Displacement**: Human collectors are priced out by MEV bots and sophisticated sniping infrastructure

### Impact Assessment

The current sniping problem creates several negative outcomes:
- **Liquidity Risk**: Large portions of accumulated $FXH reserves are immediately extractable by snipers
- **Unfair Distribution**: Early tokens meant for community distribution are concentrated among technical actors
- **Reduced Artist Revenue**: Sniped liquidity reduces the long-term value accrual for artists
- **Community Alienation**: Legitimate collectors lose confidence in fair launch mechanisms

## Specification

### Overview

The Pre-Bonding Tranche system introduces a safelist-based early access period that precedes the standard bonding curve launch. During this period, eligible 
participants can contribute $FXH with uniform pricing, creating a more equitable distribution of early tokens while maintaining the existing bonding curve 
mechanics for subsequent participants.

### Core Components

#### 1. Safelist Generation

**Primary Safelist**: Built from the fxhash airdrop snapshot, providing anti-sybil protection through established community participation.

**Additional Criteria** (TBD): The safelist can grow to include new community members. The purpose is to prevent sybil attacks on the new sale mechanism so any criteria that ensures longer term community participation would be fair. Possible criteria:
- Current $FXH holders (minimum threshold TBD)
- Active fxhash participants (minting/collecting history)
- Verified artist accounts
- Community-nominated addresses

**Safelist Properties**:
- One entry per wallet address
- No $FXH holding requirement if you are on the safelist
- Safelist entries are non-transferrable.

#### 2. Pre-Bonding Tranche Mechanics

**Fixed Allocation Model**:
- Artist sets fixed number of tokens for pre-bonding tranche (e.g., 100,000 tokens)
- These tokens are permanently removed from bonding curve supply
- Tranche always completes regardless of participation level

**Contribution Window**:
- Duration: 60 minutes (configurable per launch)
- Maximum individual contribution: $69 worth of $FXH (configurable)
- No minimum participation threshold required

**Pricing Mechanism**:
- Price determined by: `(Total $FXH Contributed) / (Fixed Token Allocation)`
- All participants receive identical per-token pricing
- Higher participation = higher initial bonding curve price
- Lower participation = lower initial bonding curve price

**Token Distribution**:
- All participants receive proportional allocation of fixed token amount
- Tokens distributed immediately before bonding curve launch
- No vesting or lock-up periods

#### 3. Curve Integration

**Supply Allocation**:
- Artist sets pre-bonding allocation (e.g., 100,000 tokens from 800,000 total)
- Remaining tokens (e.g., 700,000) available for standard bonding curve
- Bonding curve starts at price determined by pre-bonding results

**Initial Price Setting**:
- Bonding curve initial price = `(Total Pre-Bonding $FXH) / (Pre-Bonding Token Allocation)`
- Virtual liquidity adjusted to start curve at this price point
- Maintains smooth pricing progression for subsequent participants

**Liquidity Impact**:
- All pre-bonding $FXH becomes initial curve liquidity
- Higher participation creates less steep bonding curve
- Lower participation creates steeper bonding curve
- Maintained graduation mechanics at 80% of remaining supply

### Implementation Parameters

#### Configurable Settings (per launch)

```solidity
struct PreBondingConfig {
   uint256 preBondingTokenAllocation;  // e.g., 100,000 tokens
   uint256 maxContributionPerWallet;   // e.g., $69 worth of $FXH
   uint256 windowDuration;             // e.g., 3600 seconds (60 minutes)
   bool requiresArtistApproval;        // Default: false
   address[] additionalSafelist;       // Artist-specified additions
}
```

#### Default Parameters

- **Pre-Bonding Allocation**: 100,000 tokens (12.5% of 800k curve supply)
- **Max Contribution**: $69 worth of $FXH
- **Window Duration**: 60 minutes
- **Safelist Size**: ~40,000 addresses (estimated from airdrop data)

### Failure Modes & Handling

#### Low Participation

If pre-bonding tranche has minimal participation:
- **Lower Initial Price**: Bonding curve starts at lower price point
- **Steeper Curve**: Remaining tokens have more aggressive price appreciation
- **Normal Operation**: System continues to function as designed

#### High Participation

If pre-bonding tranche receives maximum participation:
- **Higher Initial Price**: Bonding curve starts at elevated price point  
- **Gentler Curve**: Remaining tokens have smoother price progression
- **Proportional Allocation**: All participants receive equal pricing

#### Excess Individual Demand

If individual contributions exceed wallet caps:
- **Automatic Limiting**: Contributions capped at maximum per wallet
- **Refund Excess**: Surplus $FXH automatically returned
- **Equal Treatment**: All participants subject to same limits

## Rationale

### Anti-Sniping Effectiveness

**Sybil Resistance**: Using established airdrop recipients creates significant barriers to large-scale sybil attacks, as these addresses have historical platform engagement.

**MEV Bot Limitation**: Fixed 60-minute window and per-wallet caps prevent automated systems from dominating allocation through speed or capital advantages.

**Community Alignment**: Prioritizing existing community members aligns with fxhash's values and supports long-term platform engagement.

### Economic Benefits

**Predictable Launch Mechanics**: Fixed token allocation eliminates uncertainty about tranche completion, providing clear expectations for both artists and participants.

**Dynamic Price Discovery**: Initial bonding curve price reflects actual community demand rather than arbitrary starting points, creating more genuine price discovery.

**Flexible Curve Steepness**: Artists can balance community distribution vs. bonding curve dynamics by adjusting pre-bonding allocation size.

**Enhanced Liquidity Distribution**: Pre-bonding participants are more likely to be long-term holders, while bonding curve steepness adjusts based on actual community engagement.

### Technical Considerations

**Gas Efficiency**: Batch processing of pre-bonding allocations reduces individual transaction costs.

**Curve Compatibility**: System maintains full compatibility with existing bonding curve mechanics and graduation processes.

**Flexibility**: Configurable parameters allow experimentation and optimization without protocol changes.

## Security Considerations

### Smart Contract Risks

**Reentrancy Protection**: All external calls implement appropriate reentrancy guards.

**Overflow Protection**: SafeMath or Solidity 0.8+ overflow protection for all arithmetic operations.

**Access Control**: Multi-signature requirement for parameter changes and emergency functions.

### Economic Attacks

**Liquidity Protection**: Ability for market makers to manipulate price not entirely mitigated, but it will require more active engagement and not a simple technical attack.

**Collusion Resistance**: Individual contribution limits reduce incentives for coordinated attacks.

**Sybil Attacks**: Maintaining a safelist will prevent a bad actor from flooding the tranche with new accounts.

**Timing Attacks**: Fixed window duration eliminates timing-based manipulation strategies.

## Backwards Compatibility

The Pre-Bonding Tranche system is designed as an additive enhancement:

- **Existing Launches**: No impact on currently active bonding curves
- **Artist Choice**: New system available as opt-in for future launches
- **Standard Curve**: Full compatibility with existing bonding curve mechanics
- **Graduation**: No changes to existing graduation processes or liquidity pool creation

## Conclusion

The Pre-Bonding Tranche system addresses the critical sniping problem affecting fxhash art coin launches while maintaining the platform's commitment to fair, transparent, and community-driven token distribution. By leveraging the existing community through safelist mechanics and providing equal access to early pricing, this system creates a more equitable launch environment that benefits artists, collectors, and the broader fxhash ecosystem.

The proposed solution is technically feasible, economically sound, and aligned with fxhash's values of supporting artists and fostering genuine community engagement around generative art projects.

---

**References:**
- [fxhash Art Coin Launches & Bonding Curves Documentation](https://docs.fxhash.xyz/fxh/art-coin-launches-and-bonding-curves)
- [fxhash Protocol Overview](https://docs.fxhash.xyz/fxh/protocol-overview)
- [GALO trading summary](https://dexscreener.com/base/0x5c2b98d8e088ddea8bd7365e72455cd64fdd69de)
