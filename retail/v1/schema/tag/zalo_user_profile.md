# zalo_user_profile

## Description

Bảng Zalo user profile info + Zalo Official Account association.

Dùng khi solution cần:
- Filter Zalo users theo demographics (gender, city, dob)
- Filter theo Zalo Official Account

## Table Name Pattern

`zalo_user_profile_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Note |
|---|---|---|
| id | String | |
| orgId | String | |
| userId | String nullable | |
| name | String nullable | |
| aliasName | String nullable | |
| phone | String | |
| email | String | |
| gender | String nullable | |
| zaloOA | String | **Zalo Official Account ID** |
| city | String nullable | |
| district | String nullable | |
| ward | String nullable | |
| address | String nullable | |
| avatar | String nullable | |
| avatar120 | String nullable | |
| avatar240 | String nullable | |
| displayName | String nullable | |
| dob | UInt16 | Date of birth (year only) |
| yob | UInt16 | Year of birth |
| createdAt | DateTime | |

## channelIdField Guidance

Query direct:
- channelType=ZaloUId → `channelIdField: "userId"`
- channelType=Phone → `channelIdField: "phone"`

## Data Quality Notes

- `userId` là nullable — có thể có records missing Zalo ID
- `dob` và `yob` cùng là year — có thể redundant