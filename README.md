[作业 poe/src/lib.rs](https://github.com/ZbkSou/substrate-node-template/blob/main/pallets/poe/src/lib.rs)

[作业 poe/src/tests.rs](https://github.com/ZbkSou/substrate-node-template/blob/main/pallets/poe/src/tests.rs)

![1662222860(1)](https://user-images.githubusercontent.com/9262343/188279970-c8941d98-67f6-4807-960e-dfe3a4445ca4.png)


```
use crate::{mock::*, Error};
use frame_support::{assert_noop, assert_ok,BoundedVec};
use super::*;
#[test]
fn create_claim_works(){
	new_test_ext().execute_with(||{
		let claim = 
		BoundedVec::<u8, <Test as Config>:: MaxBytesInHash>::try_from(vec![0,1]).unwrap();
		//assert  return ok 
		assert_ok!(PoeModule::create_claim(Origin::signed(1), claim.clone()));
		// check claim 
		let bounded_claim = 
		BoundedVec::<u8, <Test as Config>:: MaxBytesInHash>::try_from(claim.clone()).unwrap();
		assert_eq!(
			Proofs::<Test>::get(&bounded_claim),
			Some((1, frame_system::Pallet::<Test>::block_number()))
		);
	}
	)
}
#[test] 
fn create_claim_failed_when_claim_already_exist(){
	new_test_ext().execute_with(||{
		let claim = 
		BoundedVec::<u8, <Test as Config>:: MaxBytesInHash>::try_from(vec![0,1]).unwrap();
		PoeModule::create_claim(Origin::signed(1), claim.clone());
		assert_noop!(
			PoeModule::create_claim(Origin::signed(1), claim.clone()),
			Error::<Test>::	ProofAlreadyClaimed
		);
	}
	)
}
#[test] 
fn revole_claim_works(){
	new_test_ext().execute_with(||{
		let claim = 
		BoundedVec::<u8, <Test as Config>:: MaxBytesInHash>::try_from(vec![0,1]).unwrap();
	    //signed1 create claim 
		PoeModule::create_claim(Origin::signed(1), claim.clone());
		// Check others for destruction
		assert_noop!(
			PoeModule::revole_claim(Origin::signed(2), claim.clone()),
			Error::<Test>::	NotProofOwner
		);
		//signed1 for destruction
		assert_ok!(PoeModule::revole_claim(Origin::signed(1), claim.clone()));
		// check claim 
		let bounded_claim = 
		BoundedVec::<u8, <Test as Config>:: MaxBytesInHash>::try_from(claim.clone()).unwrap();
		// claim is none
		assert_eq!(
			Proofs::<Test>::get(&bounded_claim),
			None
		);
	}
	)
}

#[test] 
fn transaction_claim_works(){
	new_test_ext().execute_with(||{
		let claim = 
		BoundedVec::<u8, <Test as Config>:: MaxBytesInHash>::try_from(vec![0,1]).unwrap();
	    //signed1 create claim 
		PoeModule::create_claim(Origin::signed(1), claim.clone());
		// Check claim owner
		assert_eq!(
			Proofs::<Test>::get(&claim),
			Some((1, frame_system::Pallet::<Test>::block_number()))
		);
		//1 transaction 2
		assert_ok!(PoeModule::transaction_claim(Origin::signed(1), claim.clone(),2));
		// check claim 
		assert_eq!(
			Proofs::<Test>::get(&claim),
			Some((2, frame_system::Pallet::<Test>::block_number()))
		);
		// Check others transaction
		assert_noop!(
			PoeModule::transaction_claim(Origin::signed(1), claim.clone(),2),
			Error::<Test>::	NotProofOwner
		);
		// Check  transaction not_exist_claim
		let  not_exist_claim = 
		BoundedVec::<u8, <Test as Config>:: MaxBytesInHash>::try_from(vec![0,5]).unwrap();
		assert_noop!(
			PoeModule::transaction_claim(Origin::signed(1), not_exist_claim.clone(),2),
			Error::<Test>::	NoSuchProof
		);
	}
	)
}
```
