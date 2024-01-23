# Hot Fix Updates

## Hotfix 1  

Release Date: January 23, 2024

Notes:  

* Ticking groups with can tick when game paused are now ticking.  
* Sometimes, travelling server resulted in the system going out of sync and losing tick entries. This has been fixed.  
* There has been added safety nets to world and context checks. Some projects had mentioned that PIE was having issues. This should be fixed, but reports are welcome.  
* There was an incongruency when clearing out pending object deletions which resulted in the ticking collections removing the wrong objects until empty. This would result in any registered object not ticking anymore. This has been fixed.  
* There was an incongruency on a deprecated interface call that was resulting in issues when called. The code has been removed. Please update to the latest api to ensure no issues with deprecated functionality remain.  
